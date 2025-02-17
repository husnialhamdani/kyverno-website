---
title: "Replace Ingress Hosts"
category: Other
version: 1.9.0
subject: Ingress
policyType: "mutate"
description: >
    An Ingress may specify host names at a variety of locations in the same resource. In some cases, those host names should be modified to, for example, update domain names silently. The replacement must be done in all the fields where a host name can be specified. This policy, illustrating the use of nested foreach loops and operable in Kyverno 1.9+, replaces host names that end with `old.com` with `new.com`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/rec-req/replace-ingress-hosts/replace-ingress-hosts.yaml" target="-blank">/other/rec-req/replace-ingress-hosts/replace-ingress-hosts.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: replace-ingress-hosts
  annotations:
    policies.kyverno.io/title: Replace Ingress Hosts
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.9.0
    policies.kyverno.io/minversion: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/subject: Ingress
    policies.kyverno.io/description: >-
      An Ingress may specify host names at a variety of locations in the same resource.
      In some cases, those host names should be modified to, for example, update domain names
      silently. The replacement must be done in all the fields where a host name can be specified.
      This policy, illustrating the use of nested foreach loops and operable in Kyverno 1.9+, replaces
      host names that end with `old.com` with `new.com`.
spec:
  background: false
  rules:
    - name: replace-old-with-new
      match:
        any:
          - resources:
              kinds:
                - Ingress
      mutate:
        foreach:
          - list: request.object.spec.rules
            patchesJson6902: |-
              - path: /spec/rules/{{elementIndex}}/host
                op: replace
                value: {{replace_all('{{element.host}}', '.old.com', '.new.com')}}
          - list: request.object.spec.tls[]
            foreach:
              - list: "element.hosts"
                patchesJson6902: |-
                  - path: /spec/tls/{{elementIndex0}}/hosts/{{elementIndex1}}
                    op: replace
                    value: "{{ replace_all('{{element}}', '.old.com', '.new.com') }}"
          - list: request.object.spec.tls[]
            patchesJson6902: |-
              - path: /spec/tls/{{elementIndex}}/secretName
                op: replace
                value: "{{ replace_all('{{element.secretName}}', '.old.com', '.new.com') }}"
```
