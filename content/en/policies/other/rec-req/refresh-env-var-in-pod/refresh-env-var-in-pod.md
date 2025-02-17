---
title: "Refresh Environment Variables in Pods"
category: Other
version: 1.9.0
subject: Pod,Deployment,Secret
policyType: "mutate"
description: >
    When Pods consume Secrets or ConfigMaps through environment variables, should the contents of those source resources change, the downstream Pods are normally not aware of them. In order for the changes to be reflected, Pods must either restart or be respawned. This policy watches for changes to Secrets which have been marked for this refreshing process which contain the label `kyverno.io/watch=true` and will write an annotation to any Deployment Pod template which consume them as env vars. This will result in a new rollout of Pods which will pick up the changed values. See the related policy entitled "Refresh Volumes in Pods" for a similar reloading process when ConfigMaps and Secrets are consumed as volumes instead. Use of this policy may require providing the Kyverno ServiceAccount with permission to update Deployments.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/rec-req/refresh-env-var-in-pod/refresh-env-var-in-pod.yaml" target="-blank">/other/rec-req/refresh-env-var-in-pod/refresh-env-var-in-pod.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: refresh-env-var-in-pods
  annotations:
    policies.kyverno.io/title: Refresh Environment Variables in Pods
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod,Deployment,Secret
    kyverno.io/kyverno-version: 1.9.0
    policies.kyverno.io/minversion: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      When Pods consume Secrets or ConfigMaps through environment variables, should the contents
      of those source resources change, the downstream Pods are normally not aware of them. In order
      for the changes to be reflected, Pods must either restart or be respawned. This policy watches
      for changes to Secrets which have been marked for this refreshing process which contain the label
      `kyverno.io/watch=true` and will write an annotation to any Deployment Pod template which consume
      them as env vars. This will result in a new rollout of Pods which will pick up the changed values.
      See the related policy entitled "Refresh Volumes in Pods" for a similar reloading process when ConfigMaps
      and Secrets are consumed as volumes instead. Use of this policy may require providing the Kyverno ServiceAccount
      with permission to update Deployments.
spec:
  mutateExistingOnPolicyUpdate: false
  rules:
  - name: refresh-from-secret-env
    match:
      any:
      - resources:
          kinds:
          - Secret
          selector:
            matchLabels:
              kyverno.io/watch: "true"
    preconditions:
      all:
      - key: "{{request.operation}}"
        operator: Equals
        value: UPDATE
    mutate:
      targets:
        - apiVersion: apps/v1
          kind: Deployment
          namespace: "{{request.namespace}}"
      patchStrategicMerge:
        spec:
          template:
            metadata:
              annotations:
                corp.org/random: "{{ random('[0-9a-z]{8}') }}"
            spec:
              containers:
              - env:
                - valueFrom:
                    secretKeyRef:
                      <(name): "{{ request.object.metadata.name }}"
```
