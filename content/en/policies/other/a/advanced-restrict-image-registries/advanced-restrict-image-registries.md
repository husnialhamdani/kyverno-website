---
title: "Advanced Restrict Image Registries"
category: Other
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    In instances where a ClusterPolicy defines all the approved image registries is insufficient, more granular control may be needed to set permitted registries, especially in multi-tenant use cases where some registries may be based on the Namespace. This policy shows an advanced version of the Restrict Image Registries policy which gets a global approved registry from a ConfigMap and, based upon an annotation at the Namespace level, gets the registry approved for that Namespace.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/a/advanced-restrict-image-registries/advanced-restrict-image-registries.yaml" target="-blank">/other/a/advanced-restrict-image-registries/advanced-restrict-image-registries.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: advanced-restrict-image-registries
  annotations:
    policies.kyverno.io/title: Advanced Restrict Image Registries
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      In instances where a ClusterPolicy defines all the approved image registries
      is insufficient, more granular control may be needed to set permitted registries,
      especially in multi-tenant use cases where some registries may be based on
      the Namespace. This policy shows an advanced version of the Restrict Image Registries
      policy which gets a global approved registry from a ConfigMap and, based upon an
      annotation at the Namespace level, gets the registry approved for that Namespace.
spec:
  validationFailureAction: audit
  background: false
  rules:
    - name: validate-corp-registries
      match:
        any:
        - resources:
            kinds:
            - Pod
      context:
        # Get the value of the Namespace annotation called `corp.com/allowed-registries` and store. The value
        # must end with a wildcard. Currently assumes there is only a single registry name in the value.
        - name: nsregistries
          apiCall:
            urlPath: "/api/v1/namespaces/{{request.namespace}}"
            jmesPath: "metadata.annotations.\"corp.com/allowed-registries\" || ''"
        # Get the ConfigMap in the `default` Namespace called `clusterregistries` and store. The value of the key
        # must end with a wildcard. Currently assumes there is only a single registry name in the value.
        - name: clusterregistries
          configMap:
            name: clusterregistries
            namespace: default
      preconditions:
        any:
        - key: "{{request.operation || 'BACKGROUND'}}"
          operator: AnyIn
          value:
          - CREATE
          - UPDATE
      validate:
        message: This Pod names an image that is not from an approved registry.
        foreach:
        # Create a flattened array of all containers in the Pod.
        - list: "request.object.spec.[initContainers, ephemeralContainers, containers][]"
          deny:
            conditions:
              all:
                # Loop over every image and deny the Pod if any image doesn't match either the allowed registry in the
                # cluster ConfigMap or the annotation on the Namespace where the Pod is created.
                - key: "{{element.image}}"
                  operator: NotEquals
                  value: "{{nsregistries}}"
                - key: "{{element.image}}"
                  operator: NotEquals
                  value: "{{clusterregistries.data.registries}}"
```
