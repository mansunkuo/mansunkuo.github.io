---
title: 'Helm'
date: 2024-10-23T07:12:18+08:00
# weight: 1
# aliases: ["/first"]
tags:
  - Helm
  - Kubernetes
author: "Mansun Kuo"
showToc: true
TocOpen: true
draft: true
hidemeta: false
comments: false
# description: "Desc Text."
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: false # to disable highlightjs
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

## Lab 1: Create your first Helm chart

Let's create a folder called `k8s-summit-2024`:
```bash
REPO_NAME=k8s-summit-2024
mkdir $REPO_NAME
cd $REPO_NAME
git init
```

To create a new Helm chart called myapi, you can use the following command:
```bash
mkdir charts
helm create charts/myapi
```

This will create the following directory structure:
```bash
charts/myapi/
  ├── charts/
  ├── templates/
  ├── values.yaml
  ├── Chart.yaml
  └── ...
```

Helm generates a default structure for a new chart with several files and directories. Here's an explanation of each file and folder:

### Common files in a Helm chart
#### Chart.yaml

This file contains metadata about the Helm chart, including the chart's name, version, and other descriptive information.

{{< collapse openByDefault=true summary="charts/myapi/Chart.yaml" >}}
```yaml
apiVersion: v2
name: myapi
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
```
{{< /collapse >}}

#### values.yaml

This file defines default values for your chart. It includes configurations such as replica counts, image information, service configurations, etc. These values can be overridden when installing the chart by passing your own values.yaml.

{{< collapse openByDefault=true summary="charts/myapi/values.yaml" >}}
```yaml
# Default values for myapi.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

livenessProbe:
  httpGet:
    path: /
    port: http
readinessProbe:
  httpGet:
    path: /
    port: http

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes: []
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts: []
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}
```
{{< /collapse >}}

#### templates/
This directory contains Kubernetes manifest templates that Helm uses to generate Kubernetes resources. Helm will render these templates by substituting the values from values.yaml.

Helm already prepare some typical files like deployment, hpa, ingress, serviceand serviceaccount for you. Beside standard Kubernetes objects, there are some special files in this folders:
##### _helpers.tpl
A file for defining reusable template snippets and helper functions, commonly used for naming conventions.

{{< collapse openByDefault=true summary="charts/myapi/templates/_helpers.tpl" >}}
```go
{{/*
Expand the name of the chart.
*/}}
{{- define "myapi.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "myapi.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "myapi.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "myapi.labels" -}}
helm.sh/chart: {{ include "myapi.chart" . }}
{{ include "myapi.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "myapi.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapi.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "myapi.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "myapi.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```
{{< /collapse >}}

##### tests/test-connection.yaml
A simple test resource for verifying the chart installation by checking service connectivity.

{{< collapse openByDefault=true summary="charts/myapi/templates/tests/test_connection.yaml" >}}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapi.fullname" . }}-test-connection"
  labels:
    {{- include "myapi.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "myapi.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```
{{< /collapse >}}


##### NOTES.txt
The NOTES.txt file in a Helm chart provides helpful information or instructions to the user after the chart is installed. When a Helm chart is successfully deployed, the contents of NOTES.txt are displayed in the output.

{{< collapse openByDefault=true summary="charts/myapi/templates/NOTES.txt" >}}
```go
1. Get the application URL by running these commands:
{{- if .Values.ingress.enabled }}
{{- range $host := .Values.ingress.hosts }}
  {{- range .paths }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}{{ .path }}
  {{- end }}
{{- end }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "myapi.fullname" . }})
  export NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch its status by running 'kubectl get --namespace {{ .Release.Namespace }} svc -w {{ include "myapi.fullname" . }}'
  export SERVICE_IP=$(kubectl get svc --namespace {{ .Release.Namespace }} {{ include "myapi.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
  echo http://$SERVICE_IP:{{ .Values.service.port }}
{{- else if contains "ClusterIP" .Values.service.type }}
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "myapi.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace {{ .Release.Namespace }} $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:$CONTAINER_PORT
{{- end }}
```
{{< /collapse >}}

##### .helmignore
This file defines patterns for files and directories that should be excluded when packaging the chart (similar to .gitignore).

#### charts/
This directory is used to store any dependent charts. If your chart relies on other charts (e.g., a database), those charts can be placed here.

### Install a local Helm chart
To install the Helm chart without changing directories, you can specify the full path to the chart when running the helm install command. For example, installing a helm:
```bash
helm install myapi-release ./charts/myapi
```

{{< collapse openByDefault=false summary="helm install myapi-release ./charts/myapi" >}}
```bash
❯ helm install myapi-release ./charts/myapi
NAME: myapi-release
LAST DEPLOYED: Wed Sep 18 01:48:22 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```
{{< /collapse >}}

Here are some commands you can use to check the status and details of your Helm releases and the resources they deploy:

#### List installed Helm releases
This command lists all of the releases for a specified namespace (uses current namespace context if namespace not specified). For example:
```bash
helm list
```

{{< collapse openByDefault=false summary="helm list" >}}
```bash
❯ helm list
NAME            NAMESPACE       REVISION        UPDATED                                        STATUS          CHART           APP VERSION
myapi-release   default         1               2024-09-18 01:48:22.904366277 +0800 CST        deployed        myapi-0.1.0     1.16.0
```
{{< /collapse >}}

#### Get detailed information about a specific Helm release
This command shows the status of a named release. For example:
```bash
helm status myapi-release
```

{{< collapse openByDefault=false summary="helm status myapi-release" >}}
```bash
❯ helm status myapi-release
NAME: myapi-release
LAST DEPLOYED: Wed Sep 18 01:48:22 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```
{{< /collapse >}}

you can copy and paste the notes provides by the Helm chart and visit [http://127.0.0.1:8080](http://127.0.0.1:8080). It is a typical welcome page of nginx. 

#### Get all resources created by the Helm chart
```bash
kubectl get all -l app.kubernetes.io/instance=myapi-release
```
This command lists all Kubernetes resources that have the label `app.kubernetes.io/instance=myapi-release`. This label typically indicates that these resources are part of a specific Helm release or application instance.

{{< collapse openByDefault=false summary="kubectl get all -l app.kubernetes.io/instance=myapi-release" >}}
```bash
❯ kubectl get all -l app.kubernetes.io/instance=myapi-release
NAME                                 READY   STATUS    RESTARTS   AGE
pod/myapi-release-54b5c4d9c8-lgwvl   1/1     Running   0          2m19s

NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/myapi-release   ClusterIP   10.97.28.62   <none>        80/TCP    2m19s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myapi-release   1/1     1            1           2m19s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/myapi-release-54b5c4d9c8   1         1         1       2m19s
```
{{< /collapse >}}

## Lab 2: Modify it as an API
In this lab, we will modify this Helm chart as an API instance of [FastAPI](https://fastapi.tiangolo.com/). 

### Add an API endpoint
Let's add an API endpoint in a ConfigMap with [FastAPI]. Embedding your FastAPI application code directly into a ConfigMap is unconventional and not typically recommended for production environments, as ConfigMaps are generally used for configuration data rather than application code. We do it here to save the complexity of building our own image. Please don't do that in your formal environment.

{{< collapse openByDefault=true summary="git diff charts/myapi/values.yaml" >}}
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "myapi.fullname" . }}
  labels:
    {{- include "myapi.labels" . | nindent 4 }}
data:
  main.py: |
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    def read_root():
        return {"Hello": "World"}

    @app.get("/hello/{user}")
    def hello(user: str):
        return {"Hello": user}
```
{{< /collapse >}}

This code is a simple FastAPI web application with two endpoints: 
- **Root endpoint** (`/`): When someone accesses this URL with a GET request, the app returns a static JSON response, like `{"Hello": "World"}`. This is a simple welcome message.
- **Dynamic "hello" endpoint** (`/hello/{user}`): The URL takes a name or value (like a username) as part of the path. For example, accessing `/hello/Alice` will pass "Alice" to the function, and the app will respond with a personalized message: `{"Hello": "Alice"}`.

### Replace image and expose command and args 
Let's modify the values.yaml of out Helm chart.

{{< collapse openByDefault=true summary="git diff charts/myapi/values.yaml" >}}
```diff
diff --git a/charts/myapi/values.yaml b/charts/myapi/values.yaml
index fa277c8..288c7d3 100644
--- a/charts/myapi/values.yaml
+++ b/charts/myapi/values.yaml
@@ -5,10 +5,14 @@
 replicaCount: 1
 
 image:
-  repository: nginx
+  repository: tiangolo/uvicorn-gunicorn-fastapi
   pullPolicy: IfNotPresent
   # Overrides the image tag whose default is the chart appVersion.
-  tag: ""
+  tag: python3.11-slim
+
+# Command and args of the container
+command: ["uvicorn"]
+args: ["main:app", "--host", "0.0.0.0", "--port", "8080"]
 
 imagePullSecrets: []
 nameOverride: ""
@@ -39,6 +43,7 @@ securityContext: {}
   # runAsNonRoot: true
   # runAsUser: 1000
 
+containerPort: 8080
 service:
   type: ClusterIP
   port: 80
```
{{< /collapse >}}

In this example, `tiangolo/uvicorn-gunicorn-fastapi` is a commonly pre-built image for running FastAPI applications. You can fild all available images in [tiangolo/uvicorn-gunicorn-fastapi](https://hub.docker.com/r/tiangolo/uvicorn-gunicorn-fastapi). We choose the latest slim version.

This diff compares changes made to the `values.yaml` file of a Helm chart, specifically for the `myapi` application. Here’s what was changed:

1. Image Repository  
The image was updated to `tiangolo/uvicorn-gunicorn-fastapi`, which is a commonly pre-built image for running FastAPI applications. You can fild all available images in [tiangolo/uvicorn-gunicorn-fastapi](https://hub.docker.com/r/tiangolo/uvicorn-gunicorn-fastapi). We choose the latest slim version. This reflects a switch from serving static files (with Nginx) to running a FastAPI application.

2. Image Tag  
The tag was explicitly set to `python3.11-slim`, which ensures the application will run on this specific lightweight Python 3.11 image.

3. Command and Arguments  
The `command` and `args` fields were added. This specifies that when the container starts, it will run `uvicorn`, which is the ASGI server for FastAPI. It is configured to run the FastAPI application (`main:app`) on host `0.0.0.0` and port `8080`.

4. Container Port  
A `containerPort` field was added, setting the port inside the container to `8080`. This ensures the FastAPI application will be accessible on that port.

The changes reflect a transition from using an Nginx container to using a FastAPI container with Uvicorn. The container is now configured to run a FastAPI app at port 8080, with the image explicitly set to `python3.11-slim`.


The modified file will be:
{{< collapse openByDefault=false summary="charts/myapi/values.yaml" >}}
```yaml
# Default values for myapi.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: tiangolo/uvicorn-gunicorn-fastapi
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: python3.11-slim

# Command and args of the container
command: ["uvicorn"]
args: ["main:app", "--host", "0.0.0.0", "--port", "8080"]

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

containerPort: 8080
service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

livenessProbe:
  httpGet:
    path: /
    port: http
readinessProbe:
  httpGet:
    path: /
    port: http

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes: []
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts: []
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}

```
{{< /collapse >}}


### Mount volume and refine depoloyment
{{< collapse openByDefault=true summary="git diff charts/myapi/templates/deployment.yaml" >}}
```diff
diff --git a/charts/myapi/templates/deployment.yaml b/charts/myapi/templates/deployment.yaml
index 8012d09..ae95687 100644
--- a/charts/myapi/templates/deployment.yaml
+++ b/charts/myapi/templates/deployment.yaml
@@ -36,9 +36,11 @@ spec:
             {{- toYaml .Values.securityContext | nindent 12 }}
           image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
           imagePullPolicy: {{ .Values.image.pullPolicy }}
+          command: {{ toYaml .Values.command  | nindent 10 }}
+          args: {{ toYaml .Values.args | nindent 10 }}
           ports:
             - name: http
-              containerPort: {{ .Values.service.port }}
+              containerPort: {{ .Values.containerPort }}
               protocol: TCP
           livenessProbe:
             {{- toYaml .Values.livenessProbe | nindent 12 }}
@@ -46,12 +48,20 @@ spec:
             {{- toYaml .Values.readinessProbe | nindent 12 }}
           resources:
             {{- toYaml .Values.resources | nindent 12 }}
-          {{- with .Values.volumeMounts }}
           volumeMounts:
+            - name: app-code
+              mountPath: /app
+          {{- with .Values.volumeMounts }}
             {{- toYaml . | nindent 12 }}
           {{- end }}
-      {{- with .Values.volumes }}
       volumes:
+        - name: app-code
+          configMap:
+            name: {{ include "myapi.fullname" . }}
+            items:
+            - key: main.py
+              path: main.py
+      {{- with .Values.volumes }}
         {{- toYaml . | nindent 8 }}
       {{- end }}
       {{- with .Values.nodeSelector }}
```
{{< /collapse >}}

### Upgrade the Helm release 

Let's upgrate the Helm release as our new API:
```bash
helm upgrade --install myapi-release ./charts/myapi
```

{{< collapse openByDefault=true summary="helm upgrade --install myapi-release ./charts/myapi" >}}
```bash
❯ helm upgrade --install myapi-release ./charts/myapi
Release "myapi-release" has been upgraded. Happy Helming!
NAME: myapi-release
LAST DEPLOYED: Wed Sep 18 01:42:07 2024
NAMESPACE: default
STATUS: deployed
REVISION: 5
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```
{{< /collapse >}}

Copy and pase the command in one terminal to forward the port of API pod:

{{< collapse openByDefault=true summary="port forwarding" >}}
```bash
❯ export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
Visit http://127.0.0.1:8080 to use your application
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```
{{< /collapse >}}

Use another terminal to check your API:
{{< collapse openByDefault=true summary="check API" >}}
```bash
❯ curl localhost:8080
{"Hello":"World"}%                                                             
❯ curl localhost:8080/hello/mansun
{"Hello":"mansun"}%
```
{{< /collapse >}}

Let's check our k8s secrets:
```bash
❯ kubectl get secrets
NAME                                  TYPE                 DATA   AGE
sh.helm.release.v1.myapi-release.v1   helm.sh/release.v1   1      22m
sh.helm.release.v1.myapi-release.v2   helm.sh/release.v1   1      18m
```

`sh.helm.release.v1.myapi-release.v1` and `sh.helm.release.v1.myapi-release.v2` are secrets generated by Helm. These secrets store information about Helm releases. The suffixes .v1 and .v2 refer to different revisions of the Helm release. Each time a release is updated, a new secret is created.

The type of secret, `helm.sh/release.v1` is specific to Helm. Helm uses this type of secret to track the state of releases.

For example, it is possible to extract the manifest from the secret:
{{< collapse openByDefault=false summary="Decode Helm release" >}}
```yaml
❯ kubectl get secret sh.helm.release.v1.myapi-release.v2 -o jsonpath="{.data.release}" | base64 --decode | base64 --decode | gunzip | jq -r .manifest

---
# Source: myapi/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapi-release
  labels:
    helm.sh/chart: myapi-0.1.0
    app.kubernetes.io/name: myapi
    app.kubernetes.io/instance: myapi-release
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
automountServiceAccountToken: true
---
# Source: myapi/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapi-release
  labels:
    helm.sh/chart: myapi-0.1.0
    app.kubernetes.io/name: myapi
    app.kubernetes.io/instance: myapi-release
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
data:
  main.py: |
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    def read_root():
        return {"Hello": "World"}

    @app.get("/hello/{user}")
    def hello(user: str):
        return {"Hello": user}
---
# Source: myapi/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapi-release
  labels:
    helm.sh/chart: myapi-0.1.0
    app.kubernetes.io/name: myapi
    app.kubernetes.io/instance: myapi-release
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: myapi
    app.kubernetes.io/instance: myapi-release
---
# Source: myapi/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapi-release
  labels:
    helm.sh/chart: myapi-0.1.0
    app.kubernetes.io/name: myapi
    app.kubernetes.io/instance: myapi-release
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: myapi
      app.kubernetes.io/instance: myapi-release
  template:
    metadata:
      labels:
        helm.sh/chart: myapi-0.1.0
        app.kubernetes.io/name: myapi
        app.kubernetes.io/instance: myapi-release
        app.kubernetes.io/version: "1.16.0"
        app.kubernetes.io/managed-by: Helm
    spec:
      serviceAccountName: myapi-release
      securityContext:
        {}
      containers:
        - name: myapi
          securityContext:
            {}
          image: "tiangolo/uvicorn-gunicorn-fastapi:python3.11-slim"
          imagePullPolicy: IfNotPresent
          command: 
          - uvicorn
          args: 
          - main:app
          - --host
          - 0.0.0.0
          - --port
          - "8080"
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
          volumeMounts:
            - name: app-code
              mountPath: /app
      volumes:
        - name: app-code
          configMap:
            name: myapi-release
            items:
            - key: main.py
              path: main.py
```
{{< /collapse >}}

It is a templated k8s objects of the release. That is why Helm can rollback our alpplication to any release. 

Let's try to rollback to the original nginx version:
```bash
❯ helm rollback myapi-release 1
Rollback was a success! Happy Helming!
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                STATUS          CHART           APP VERSION
myapi-release   default         3               2024-09-18 02:34:03.72739722 +0800 CST deployed        myapi-0.1.0     1.16.0
```

Rollback to the FastAPI version:
```bash
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                        STATUS          CHART           APP VERSION
myapi-release   default         4               2024-09-18 02:38:42.936136839 +0800 CST        deployed        myapi-0.1.0     1.16.0
```

It populate a new revision from the revision you assigned.

## Lab 3: Why my secret is not updated
Let's add a random passcode in out API.

### Add a random secret and encode it
Helm provides a lot of handy [template functions and pipelines](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/). You can find more useful template functions in [Template Function List](https://helm.sh/docs/chart_template_guide/function_list/). For example, here is a k8s secret object with a random 10-digit passcode. We also pipe it into another function that encode a string into base64 encoding. 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "myapi.fullname" . }}
  labels:
    {{- include "myapi.labels" . | nindent 4 }}
type: Opaque
data:
  passcode: {{ randAlphaNum  10 | b64enc }}  # Generate a random 10-digit passcode and encode it in base64
```

### Mount the secret on the deployment
Let's mount our secret as an environment variable on the deployment. No special trick here, just remember to reuse template function, fullname which generated by `helm create`.

```diff
diff --git a/charts/myapi/templates/deployment.yaml b/charts/myapi/templates/deployment.yaml
index ae95687..172c1b5 100644
--- a/charts/myapi/templates/deployment.yaml
+++ b/charts/myapi/templates/deployment.yaml
@@ -38,6 +38,12 @@ spec:
           imagePullPolicy: {{ .Values.image.pullPolicy }}
           command: {{ toYaml .Values.command  | nindent 10 }}
           args: {{ toYaml .Values.args | nindent 10 }}
+          env:
+            - name: PASSCODE
+              valueFrom:
+                secretKeyRef:
+                  name: {{ include "myapi.fullname" . }}
+                  key: passcode
           ports:
             - name: http
               containerPort: {{ .Values.containerPort }}
```

### Consume the environment variable
```diff
diff --git a/charts/myapi/templates/configmap.yaml b/charts/myapi/templates/configmap.yaml
index 9d96063..0449698 100644
--- a/charts/myapi/templates/configmap.yaml
+++ b/charts/myapi/templates/configmap.yaml
@@ -6,13 +6,14 @@ metadata:
     {{- include "myapi.labels" . | nindent 4 }}
 data:
   main.py: |
+    import os
     from fastapi import FastAPI
 
     app = FastAPI()
 
     @app.get("/")
     def read_root():
-        return {"Hello": "World"}
+        return {"Hello": f"Your pass code is {os.environ.get('PASSCODE')}"}
 
     @app.get("/hello/{user}")
     def hello(user: str):
```

### What's wrong
Let's upgrade our Helm release and check what is inside the secret: 
```bash
helm upgrade --install myapi-release ./charts/myapi
kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
```

You can execute above code snippet multiple times. The secret will changes in every release, but your API will still with the oldest secret.

{{< collapse openByDefault=false summary="Helm upgrade and check secret" >}}
```bash
❯ helm upgrade --install myapi-release ./charts/myapi
Release "myapi-release" has been upgraded. Happy Helming!
NAME: myapi-release
LAST DEPLOYED: Wed Sep 25 22:57:26 2024
NAMESPACE: default
STATUS: deployed
REVISION: 14
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
❯ kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
XZB3yxL0Hv%                                                                    
❯ helm upgrade --install myapi-release ./charts/myapi
Release "myapi-release" has been upgraded. Happy Helming!
NAME: myapi-release
LAST DEPLOYED: Wed Sep 25 23:00:52 2024
NAMESPACE: default
STATUS: deployed
REVISION: 15
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
❯ kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
RiNr1OQLMd%
```
{{< /collapse >}}



Changes to ConfigMap or Secret objects that are referenced in a pod spec do not trigger a pod rollout on their own, even if the underlying data changes. This is because Kubernetes doesn't watch for changes to those resources by default. Let's add a little trick to make Helm [automatically roll deployments](https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments):


```diff
diff --git a/charts/myapi/templates/deployment.yaml b/charts/myapi/templates/deployment.yaml
index 172c1b5..1e324c8 100644
--- a/charts/myapi/templates/deployment.yaml
+++ b/charts/myapi/templates/deployment.yaml
@@ -13,8 +13,11 @@ spec:
       {{- include "myapi.selectorLabels" . | nindent 6 }}
   template:
     metadata:
-      {{- with .Values.podAnnotations }}
       annotations:
+        # https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments
+        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
+        checksum/secret: {{ include (print $.Template.BasePath "/secret.yaml") . | sha256sum }}
+      {{- with .Values.podAnnotations }}
         {{- toYaml . | nindent 8 }}
       {{- end }}
       labels:
```


Helm will roll out the pod when the content of the deployment spec or the checksum annotations for ConfigMap or Secret change. By including the checksums of the ConfigMap and Secret, the deployment will automatically be rolled out whenever these files change. This ensures that the changes in configuration or secrets trigger a new pod deployment.

The whole deployment config will look like this:
{{< collapse openByDefault=false summary="charts/myapi/templates/deployment.yaml" >}}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapi.fullname" . }}
  labels:
    {{- include "myapi.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapi.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        # https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        checksum/secret: {{ include (print $.Template.BasePath "/secret.yaml") . | sha256sum }}
      {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "myapi.labels" . | nindent 8 }}
        {{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "myapi.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command: {{ toYaml .Values.command  | nindent 10 }}
          args: {{ toYaml .Values.args | nindent 10 }}
          env:
            - name: PASSCODE
              valueFrom:
                secretKeyRef:
                  name: {{ include "myapi.fullname" . }}
                  key: passcode
          ports:
            - name: http
              containerPort: {{ .Values.containerPort }}
              protocol: TCP
          livenessProbe:
            {{- toYaml .Values.livenessProbe | nindent 12 }}
          readinessProbe:
            {{- toYaml .Values.readinessProbe | nindent 12 }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          volumeMounts:
            - name: app-code
              mountPath: /app
          {{- with .Values.volumeMounts }}
            {{- toYaml . | nindent 12 }}
          {{- end }}
      volumes:
        - name: app-code
          configMap:
            name: {{ include "myapi.fullname" . }}
            items:
            - key: main.py
              path: main.py
      {{- with .Values.volumes }}
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```
{{< /collapse >}} 

## Lab 4: Add a Helm dependency
[Helm dependency](https://helm.sh/docs/helm/helm_dependency/) manage the dependencies of a chart. Helm charts store their dependencies in 'charts/'. For chart developers, it is often easier to manage dependencies in 'Chart.yaml' which declares all dependencies.

The dependency commands operate on that file, making it easy to synchronize between the desired dependencies and the actual dependencies stored in the 'charts/' directory.

The [Bitnami Library for Kubernetes](https://github.com/bitnami/charts) is a Helm repository with various pre-packaged Kubernetes resources that make it easier to deploy common open-source applications and infrastructure components in Kubernetes clusters. There is a special chart, [Bitnami Common Library Chart](https://github.com/bitnami/charts/tree/main/bitnami/common), which grouping common logic between Bitnami charts. Let's add it into our Helm chart.

### Add a dependency
```diff
diff --git a/charts/myapi/Chart.yaml b/charts/myapi/Chart.yaml
index e1991d4..4b34b58 100644
--- a/charts/myapi/Chart.yaml
+++ b/charts/myapi/Chart.yaml
@@ -22,3 +22,8 @@ version: 0.1.0
 # follow Semantic Versioning. They should reflect the version the application is using.
 # It is recommended to use it with quotes.
 appVersion: "1.16.0"
+
+dependencies:
+  - name: common
+    version: 2.x.x
+    repository: oci://registry-1.docker.io/bitnamicharts
\ No newline at end of file
```

`x` means latest for major, minor, or patch of a semantic versioning.

### Build dependency
Let's refresh the Helm dependency based on Chart.yaml
```bash
❯ helm dependency update charts/myapi
Saving 1 charts
Downloading common from repo oci://registry-1.docker.io/bitnamicharts
Pulled: registry-1.docker.io/bitnamicharts/common:2.22.0
Digest: sha256:7e1d75e30a368544c724e480b8f375f702ecdd6933634a852b760660c6cbd588
Deleting outdated charts
```

This command will create a `Chart.lock` file. You can use `helm dependency build` to reconstruct a chart's dependencies to the state specified in the lock file. Here is a summary of the difference between `helm dependency build` and `helm dependency update`:

| **Command**              | **Works With**       | **Downloads Dependencies**        | **Updates Chart.lock**         | **Use Case**                                        |
|--------------------------|----------------------|-----------------------------------|-------------------------------|----------------------------------------------------|
| `helm dependency build`   | `Chart.lock`         | Based on the locked versions      | No                            | Install exact versions of dependencies as specified in the `Chart.lock` file for reproducibility. |
| `helm dependency update`  | `Chart.yaml`         | Based on the latest matching versions in `Chart.yaml` | Yes                           | Update dependencies to the latest matching versions as specified in `Chart.yaml`, and regenerate the `Chart.lock` file. |

It also generate a `charts/` forder which contains all dependencies for the chart. Starting from 2.2.0, repository can be defined as the path to the directory of the dependency charts stored locally. The path should start with a prefix of "file://". For example:

```yaml
# Chart.yaml
dependencies:
- name: nginx
  version: "1.2.3"
  repository: "file://../dependency_chart/nginx"
```

If the dependency chart is retrieved locally, it is not required to have the repository added to helm by "helm add repo". Version matching is also supported for this case.

In our case, we are using an external dependency. We don't need to add it into our git repository. Let's add an .gitignore for it:
```bash
curl -o .gitignore https://raw.githubusercontent.com/bitnami/charts/refs/heads/main/.gitignore
``` 

{{< collapse openByDefault=false summary=".gitignore" >}}
```git
*.tgz
/.idea/*
.vscode
.DS_Store
```
{{< /collapse >}} 

### OCI-based registries and traditional chart repository
Beginning in Helm 3, you can use container registries with [OCI (Open Container Initiative)](https://opencontainers.org/) support to store and share chart packages. Beginning in Helm v3.8.0, OCI support is enabled by default. You don't need to use helm repo add for an OCI (Open Container Initiative) registry (oci://) because Helm interacts with OCI registries in a different way than with traditional Helm chart repositories.

Key Differences Between OCI Registries and Helm Chart Repositories:
1. Traditional Helm Repositories:
- Traditional Helm repositories (e.g., https://charts.example.com) use the helm repo add command to register the repository URL with Helm. This allows Helm to search, fetch, and install charts from the repository using a simple chart name and version.
- Repositories store a index.yaml file that acts as a catalog of all the charts, which Helm uses to fetch charts by name.
2. OCI Registries:
- OCI registries are more akin to Docker image registries (e.g., Docker Hub), where charts are stored as OCI artifacts.
- You don’t need an index file (like index.yaml) or a repository registration step (helm repo add). Instead, you can directly interact with the OCI registry using commands like helm pull, helm push, and helm install with the oci:// scheme.

With OCI registries, Helm interacts directly with charts using the oci:// scheme, bypassing the need for helm repo add and index.yaml files. This is more similar to working with Docker images than traditional Helm chart repositorie


## Referernces
- [Chart Releaser Action to Automate GitHub Page Charts](https://helm.sh/docs/howto/chart_releaser_action/)
- [Artifact Hub - Helm charts repositories](https://artifacthub.io/docs/topics/repositories/helm-charts/)
- https://helm.sh/docs/topics/chart_repository/
- https://helm.sh/docs/topics/registries/