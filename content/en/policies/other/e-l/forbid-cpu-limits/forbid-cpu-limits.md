---
title: "Forbid CPU Limits"
category: Other
version: 
subject: Pod
policyType: "validate"
description: >
    Setting of CPU limits is a debatable poor practice as it can result, when defined, in potentially starving applications of much-needed CPU cycles even when they are available. Ensuring that CPU limits are not set may ensure apps run more effectively. This policy forbids any container in a Pod from defining CPU limits.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/e-l/forbid-cpu-limits/forbid-cpu-limits.yaml" target="-blank">/other/e-l/forbid-cpu-limits/forbid-cpu-limits.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: forbid-cpu-limits
  annotations:
    policies.kyverno.io/title: Forbid CPU Limits
    policies.kyverno.io/category: Other
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.10.0
    kyverno.io/kubernetes-version: "1.26"
    policies.kyverno.io/description: >-
      Setting of CPU limits is a debatable poor practice as it can result, when defined, in potentially starving
      applications of much-needed CPU cycles even when they are available. Ensuring that CPU limits are not
      set may ensure apps run more effectively. This policy forbids any container in a Pod from defining CPU limits.
spec:
  background: true
  validationFailureAction: Enforce
  rules:
  - name: check-cpu-limits
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: Containers may not define CPU limits.
      pattern:
        spec:
          containers:
          - (name): "*"
            =(resources):
              =(limits):
                X(cpu): null

```
