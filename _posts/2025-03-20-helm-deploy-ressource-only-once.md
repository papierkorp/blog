---
layout: post
title: How to deploy a Kubernetes Ressource (configmap) only once and keep the data
date: 2025-03-20
tags: kubernetes howto devops helm
subtitle: 'Deploy a Configmap only at the install but keep it after each upgrade.'
comments_id: 1
---

Today I hade the problem that i wanted to install a configmap only once but keep it after each upgrade, which wasnt as easy i thought before.
Since it took my quite some time to figure this one out, here a short how to:

# Only run once

At first I fell for the mistake of installing the Ressource only once and the leaving it at that like this:

```yaml
{{- if .Release.IsInstall }}
kind: ConfigMap
apiVersion: v1
metadata:
{{- end }}
---
{{- if le .Release.Revision 1  }}
kind: ConfigMap
apiVersion: v1
metadata:
{{- end }}
```

Unfortunately Helm isnt working like this. Instead helm validates my expression as true and doesnt deploy the configmap. But then, since the configmap is no longer deployed its deleted by helm afterwards since its orphaned.

This lead me to the second approach of keeping the data.

# Keep the data

As far as I found out there are 2 ways to keep the data in helm:

- [helm lookup](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/#using-the-lookup-function)
- [helm resource-policy Annotation](https://helm.sh/docs/howto/charts_tips_and_tricks/#tell-helm-not-to-uninstall-a-resource)

## lookup

This is the solution i settled with, i havent tested it yet with huge configmaps but I´m confident it will work as well =).

```yaml
{{- $existing_cm := (lookup "v1" "ConfigMap" .Release.Namespace "example-conf") }}
kind: ConfigMap
apiVersion: v1
metadata:
    name: example-conf
    namespace: {{ .Values.namespace }}
data:
{{- if $existing_cm }}
{{- with $existing_cm.data }}
{{ toYaml . | nindent 2 }}
{{- end }}
{{- else }}
  app.yml: |
    use-default: true
{{- end }}
```
## annotation

The disadvantage of this solution is: The configmap is kept even after a helm uinstall and will no longer be managed by helm. Of course it would be deleted after the namespace is removed.

```yaml
{{- if .Release.IsInstall }}
kind: ConfigMap
apiVersion: v1
metadata:
    name: example-conf
    namespace: {{ .Values.namespace }}
    annotations:
        "helm.sh/resource-policy": keep
data:
  app.yml: |
    use-default: true
{{- end }}
```

