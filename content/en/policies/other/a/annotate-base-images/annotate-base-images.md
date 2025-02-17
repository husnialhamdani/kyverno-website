---
title: "Annotate Base Images"
category: Other
version: 1.7.0
subject: Pod
policyType: "mutate"
description: >
    A base image used to construct a container image is not accessible by any Kubernetes component and not a field in a Pod spec as it must be fetched from a registry. Having this information available in the resource referencing the containers helps to provide a clearer understanding of its contents. This policy adds an annotation to a Pod or its controllers with the base image used for each container if present in an OCI annotation.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/a/annotate-base-images/annotate-base-images.yaml" target="-blank">/other/a/annotate-base-images/annotate-base-images.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: annotate-base-images
  annotations:
    policies.kyverno.io/title: Annotate Base Images
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    pod-policies.kyverno.io/autogen-controllers: none
    kyverno.io/kyverno-version: 1.7.0
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      A base image used to construct a container image is not accessible
      by any Kubernetes component and not a field in a Pod spec as it must
      be fetched from a registry. Having this information available in the resource
      referencing the containers helps to provide a clearer understanding of
      its contents. This policy adds an annotation to a Pod or its controllers
      with the base image used for each container if present in an OCI annotation.
spec:
  rules:
  - name: mutate-base-image
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    mutate:
      foreach:
      - list: "request.object.spec.containers"
        context:
        - name: imageData
          imageRegistry: 
            reference: "{{ element.image }}"
        - name: basename
          variable:
            jmesPath: imageData.manifest.annotations."org.opencontainers.image.base.name"
            default: ''
        patchesJson6902: |-
          - path: "/metadata/annotations/kyverno.io~1baseimages{{elementIndex}}"
            op: add
            value: "{{basename}}"
```
