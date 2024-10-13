---
title: '為美好的世界獻上 Helm Chart'
date: 2024-10-23T07:12:18+08:00
# weight: 1
# aliases: ["/first"]
tags:
  - Helm
  - Kubernetes
  - 工作坊
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
Summary: |
  體驗工作坊 - Kubernetes Summit 2024
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://ccmsassets.ithome.com.tw/2023/8/4/df264c1c-5add-4173-a8c6-71330ff65537.jpg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

這篇文章是這次我在台灣 [Kubnetes Summit 2024](https://k8s.ithome.com.tw/2024/workshop-page/3261) 所帶領的工作坊，在這個實戰工作坊中，我們會介紹一個標準的 Helm Chart 的目錄架構以及裡面各個元件的基本設定，帶著您從無到有建立一個自己的 Helm Chart，使用 Helm Template 以及 Helm dependency 寫出容易使用以及可擴展的 Helm Chart，並實際使用 GitHub Page 以及 GitHub Action 讓您最新版本的 Helm Chart 可以透過 Helm Repo更容易分享給別人，最後會再讓大家實際去使用自己或是其他學員所包好的 Helm Chart 來部署在自己的 Kubernetes 叢集，順便熟悉一些重要的指令以及使用上的一些小技巧。文章很長，還請善用目錄來幫您快速跳轉到你想去的地方。

## 練習一: 創建您的第一個 Helm Chart

讓我們新增一個名為 `k8s-summit-2024` 的資料夾：
```bash
REPO_NAME=k8s-summit-2024
mkdir $REPO_NAME
cd $REPO_NAME
git init
```

要新增一個名為 myapi 的 Helm Chart，可以使用以下命令：
```bash
mkdir charts
helm create charts/myapi
```

Helm 會生成一個新 Chart 的默認結構，其中包含幾個文件和目錄：
```bash
.
└── charts
    └── myapi
        ├── charts
        ├── templates
        │   ├── NOTES.txt
        │   ├── _helpers.tpl
        │   ├── deployment.yaml
        │   ├── hpa.yaml
        │   ├── ingress.yaml
        │   ├── service.yaml
        │   ├── serviceaccount.yaml
        │   └── tests
        │       └── test-connection.yaml
        ├── .helmignore
        ├── Chart.yaml
        └── values.yaml
```

以下是每個檔案和資料夾的簡介：

### Helm chart 的常見檔案
#### Chart.yaml

這個文件包含有關 Helm Chart 的後設資料，包括圖表的名稱、版本和其他描述性資訊。

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

這個文件定義了您的 Helm chart 的默認值。在安裝圖表時，您可以通過傳遞自己的 values.yaml 文件來覆蓋這些值。

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
這個目錄包含 Helm 用來生成 Kubernetes 資源的 Kubernetes 清單模板。Helm 會通過替換 values.yaml 中的值來渲染這些模板。
Helm 已經為您準備了一些典型的檔案，比如 deployment、hpa、ingress、service 和 serviceaccount。除了標準的 Kubernetes 物件外，該文件夾中還有一些特殊的檔案：

##### _helpers.tpl
用於定義可重複使用模板片段和輔助函數的檔案，通常用於命名規範。

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
一個簡單的測試資源，用於通過檢查服務連線性來驗證安裝結果。

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
Helm chart 中的 NOTES.txt 文件在圖表安裝後為用戶提供有用的信息或說明。當 Helm chart 成功部署時，NOTES.txt 的內容會顯示在輸出中。

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
這個檔案定義了在打包 chart 時應排除的檔案和目錄的模式（類似於 .gitignore）。

#### charts/
這個目錄用來儲存任何依賴的 charts。如果您的 chart 依賴於其他 charts（例如，資料庫），這些 charts 可以放在這裡。


### 安裝本地的 local Helm chart
要在不更改目錄的情況下安裝 Helm chart，您可以在執行 helm install 命令時指定 chart 的完整路徑。例如，安裝一個名為 `myapi-release` 的 helm release：
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

以下是一些您可以使用的命令，用於檢查您的 Helm 釋出的狀態和詳細資訊，以及它們所部署的資源：

#### 列出已安裝的 Helm releases
這個命令會列出指定命名空間的所有 releases（如果未指定命名空間，則使用當前的命名空間）。例如：
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

#### 獲取有關特定 Helm release 的詳細資訊
此命令顯示指定名稱釋出的狀態。例如：
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

您可以複製貼上 Helm chart 提供的註解，然後訪問 [http://127.0.0.1:8080](http://127.0.0.1:8080)。這是一個大家都很熟悉的 nginx 歡迎頁面。

#### 獲取由 Helm chart 創建的所有資源
```bash
kubectl get all -l app.kubernetes.io/instance=myapi-release
```
此命令列出所有具有 `app.kubernetes.io/instance=myapi-release` 這個標籤的 Kubernetes 資源。這個標籤通常表示這些資源是特定 Helm release 或應用實例的一部分。

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

## 練習二：將其修改為 API
在這個練習中，我們將修改這個 Helm chart，以創建 [FastAPI](https://fastapi.tiangolo.com/) 的 API 實例。


### 添加 API 端點
讓我們在 ConfigMap 中添加 API 端點。我們一般不會將 FastAPI 的程式碼直接嵌入到 ConfigMap ， ConfigMap 通常用於配置設定，而不是程式碼。我們這樣做是為了在這個課程中省略構建我們自己的容器的過程。請不要直接在正式環境中這樣做。

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

這段程式碼是一個簡單的 FastAPI 網頁應用程式，具有兩個端點：
- **根端點** (`/`): 當有人以 GET 請求訪問此 URL 時，應用程式會返回靜態 JSON 響應，如 `{"Hello": "World"}`。這是一個簡單的歡迎消息。
- **動態 "hello" 端點** (`/hello/{user}`): 該 URL 將名稱或值（如用戶名）作為路徑的一部分。例如，訪問 `/hello/Mansun` 將把 "Mansun" 傳遞給函數，應用程式將回覆：`{"Hello": "Mansun"}`。

### 替換容器映像檔並設定命令和參數
讓我們修改我們的 Helm chart 的 values.yaml。

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

這個差異比較了對 Helm chart 的 values.yaml 檔案所做的更改：

1. 映像檔倉庫  
映像檔已更新為 `tiangolo/uvicorn-gunicorn-fastapi`，這是一個常見的預建映像檔，用於運行 FastAPI 應用程式。您可以在 tiangolo/[tiangolo/uvicorn-gunicorn-fastapi](https://hub.docker.com/r/tiangolo/uvicorn-gunicorn-fastapi) 中找到所有可用的映像檔。我們選擇最新的精簡版本。這個變動讓我們從使用 Nginx 服務靜態檔案，轉變為運行 FastAPI 應用程式。

2. 映像標籤  
映像標籤已明確設置為 `python3.11-slim`，這確保應用程式將在這個特定的輕量級 Python 3.11 映像上運行。

3. 命令和參數  
添加了 `command` 和 `args` 欄位。這指定當容器啟動時，將運行 `uvicorn`，這是 FastAPI 的 ASGI 伺服器。它被配置為在主機 `0.0.0.0` 和通訊埠 `8080` 上運行 FastAPI 應用程式 (`main:app`)。

4. 容器通訊埠  
新增 `containerPort` 欄位，將容器內的通訊埠設置為 `8080`。這確保 FastAPI 應用程式可以在該通訊埠上訪問。

修改後的檔案：
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


### 掛載 volume 及調整部署
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

以下是主要變更的解釋：
1. 新增 Command 和 Args  
在容器規範中新增 `command` 和 `args` 欄位，允許您通過 Helm values (`.Values.command` 和 `.Values.args`) 覆蓋預設的容器進入點和參數。
2. 通訊埠參考更新  
更改了通訊埠值的來源。現在使用 `.Values.containerPort`，而不是 `.Values.service.port`。
3. 掛載應用程式碼 Volume  
將名為 `app-code` 的卷掛載到容器內的 `/app` 目錄。
4. 應用程式碼 Volume 定義  
定義了一個名為 `app-code` 的 Volume，引用了包含 `main.py` 的 ConfigMap。此 ConfigMap 通過 `myapi.fullname` 輔助模板引入，並將其中的 `main.py` 鍵掛載為容器中的 `main.py`。
5. 重新組織 volumes 和 volumeMounts 區塊  
引入了新的掛載 (`app-code`)，並且現有的 volumeMounts and volumes 仍然可以通過 `with` 指令增加 `.Values.volumeMounts` 和 `.Values.volumes` 中的任何其他值。


### 升級 Helm release

升級 Helm release，讓我們的新 API 可供使用:
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

在一個終端機中複製並貼上指令，將 API Pod 的通訊埠暴露出來：

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

使用另一個終端機檢查你的 API：
{{< collapse openByDefault=true summary="check API" >}}
```bash
❯ curl localhost:8080
{"Hello":"World"}%                                                             
❯ curl localhost:8080/hello/mansun
{"Hello":"mansun"}%
```
{{< /collapse >}}

讓我們檢查一下我們的 k8s secrets：
```bash
❯ kubectl get secrets
NAME                                  TYPE                 DATA   AGE
sh.helm.release.v1.myapi-release.v1   helm.sh/release.v1   1      22m
sh.helm.release.v1.myapi-release.v2   helm.sh/release.v1   1      18m
```

`sh.helm.release.v1.myapi-release.v1` 和 `sh.helm.release.v1.myapi-release.v2` 是由 Helm 生成的 secrets。這些 secrets 用來儲存 Helm release 的資訊。後綴 .v1 和 .v2 分別代表不同版本的 Helm release。每次更新 release 時，都會創建一個新的 secret。

secret 的類型 helm.sh/release.v1 是 Helm 特有的類型。Helm 使用這種 secret 來追蹤 release 的狀態。

例如，可以從 secret 中提取出 manifest：
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

這是一個針對 release 的模板化 k8s 物件。這也是為什麼 Helm 可以將我們的應用程式回滾到任何 release。

讓我們嘗試回滾到原本的 nginx 版本：
```bash
❯ helm rollback myapi-release 1
Rollback was a success! Happy Helming!
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                STATUS          CHART           APP VERSION
myapi-release   default         3               2024-09-18 02:34:03.72739722 +0800 CST deployed        myapi-0.1.0     1.16.0
```

回滾到 FastAPI 版本：
```bash
❯ helm rollback myapi-release 2
Rollback was a success! Happy Helming!
❯ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                        STATUS          CHART           APP VERSION
myapi-release   default         4               2024-09-18 02:38:42.936136839 +0800 CST        deployed        myapi-0.1.0     1.16.0
```

它會從您指定的 revision 去長出一個新的 revision。

## 練習三：為什麼我的 secret 沒有更新
讓我們在 API 中添加一個隨機的通關密碼。

### 添加一個隨機密鑰並對其進行編碼
Helm 提供了許多方便的 [模板函數和管線](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)。您可以在 [模板函數列表](https://helm.sh/docs/chart_template_guide/function_list/) 中找到更多有用的模板函數。例如，這裡有一個包含隨機 10 位數密碼的 k8s secret。我們將它經由管線傳遞到另一個函數，將字符串編碼為 base64 編碼。

{{< collapse openByDefault=true summary="charts/myapi/templates/secret.yaml" >}}
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
{{< /collapse >}}

### 在部署中掛載密鑰
讓我們將密鑰掛載為部署中的環境變數。這裡沒有特別的技巧，只需要記得重複使用先前透過 `helm create` 生成的模板函數 `myapi.fullname`。

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

### 使用環境變數
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

### 問題在哪
讓我們升級我們的 Helm release 並檢查該 secret 的內容：
```bash
helm upgrade --install myapi-release ./charts/myapi
kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
```

您可以多次執行上述程式碼片段。每次執行時， secret 將會改變，但您的 API 仍將使用最舊的 secret 。

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

對於在 Pod 規範中引用的 ConfigMap 或 Secret 物件，若其內容發生變更，儘管底層資料有所改變，並不會自動觸發 Pod 的滾動更新。這是因為 Kubernetes 預設並不會監視這些資源的變化。我們來使用一個小技巧，讓 Helm 可以 [自動滾動更新部署](https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments)：

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


當部署規範的內容或 ConfigMap 或 Secret 的 checksum 標註發生變更時，Helm 將會滾動更新 Pod。透過包含 ConfigMap 和 Secret 的 checksum，當這些檔案變更時，部署將自動進行滾動更新。這確保了設定檔或 secrets 的變更會觸發新的 Pod 部署。

整個部署配置將如下所示：
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

## 練習四：新增 Helm dependency
[Helm dependency](https://helm.sh/docs/helm/helm_dependency/) 用來管理一個 Helm chart 所依賴的其他 Helm charts 。 Helm charts 將其儲存在 `charts/` 資料夾中。對於 chart 開發者來說，直接管理 `Chart.yaml` 中的依賴通常更為簡單。

`helm dependency` 會作用於該檔案，使得在所需的依賴和實際存放在 `charts/` 資料夾中的其他 Helm charts 之間的同步變得容易。

[Bitnami Library for Kubernetes](https://github.com/bitnami/charts) 是一個 Helm repository ，提供各種預先打包的 Kubernetes 資源，使在 Kubernetes 叢集上部署常見的開源應用程式和基礎設施組件變得更容易。這裡有一個特殊的 chart， Bitnami Common Library Chart](https://github.com/bitnami/charts/tree/main/bitnami/common) ，它將 Bitnami charts 之間的共通邏輯進行分組。讓我們將它添加到我們的 Helm chart 中。

### 新增一個 dependency
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

`x` 代表 [Semantic Versioning](https://semver.org/) 中的主要、次要或修補版本的最新版本。

### 建立 dependency
讓我們根據 `Chart.yaml` 刷新 Helm dependency：

```bash
❯ helm dependency update charts/myapi
Saving 1 charts
Downloading common from repo oci://registry-1.docker.io/bitnamicharts
Pulled: registry-1.docker.io/bitnamicharts/common:2.22.0
Digest: sha256:7e1d75e30a368544c724e480b8f375f702ecdd6933634a852b760660c6cbd588
Deleting outdated charts
```

此命令將產生一個 `Chart.lock` 檔案。您可以使用 `helm dependency build` 來重建 Helm chart 所依賴的其他 charts，以符合鎖定檔案中指定的狀態。以下是 `helm dependency build` 和 `helm dependency update` 之間差異的總結：

| **命令**              | **適用於**       | **下載相依**        | **更新 Chart.lock**         | **使用情境**                                        |
|--------------------------|----------------------|-----------------------------------|-------------------------------|----------------------------------------------------|
| `helm dependency build`   | `Chart.lock`         | 根據鎖定的版本      | 否                            | 	安裝 `Chart.lock` 檔案中指定的依賴項的確切版本，以確保可重現性。 |
| `helm dependency update`  | `Chart.yaml`         | 根據 `Chart.yaml` 中可得到的最新的版本 | 是                           | 根據 `Chart.yaml` 中指定的最新匹配版本更新並重新生成 `Chart.lock` 檔案。 |

上述指令還會生成一個 `charts/` 資料夾，其中包含該 chart 的所有依賴項。從 Helm 2.2.0 開始，可以將 repository 定義為儲存在本地的 charts 的目錄路徑。該路徑應以 "file://" 的前綴開始。例如：


```yaml
# Chart.yaml
dependencies:
- name: nginx
  version: "1.2.3"
  repository: "file://../dependency_chart/nginx"
```

如果依賴的 chart 是從本地的檔案或使用 OCI-based registries ，則不需要通過 `helm add repo` 將 repository 添加到 helm 之中。

在我們的例子中，我們使用的是外部依賴。我們不需要將 `*.tgz` 檔案添加到我們的 git 存儲庫中。讓我們為此添加一個 .gitignore：
```bash
curl -o .gitignore https://raw.githubusercontent.com/bitnami/charts/refs/heads/main/.gitignore
``` 

{{< collapse openByDefault=true summary=".gitignore" >}}
```git
*.tgz
/.idea/*
.vscode
.DS_Store
```
{{< /collapse >}} 

### OCI-based registries 和傳統的 chart repository
自 Helm 3 起，您可以使用支援 [OCI (Open Container Initiative)](https://opencontainers.org/) 的 container registries 來儲存和分享 chart 套件。從 Helm v3.8.0 開始，OCI 支援預設已啟用。您不需要對 OCI (Open Container Initiative) 註冊表（`oci://`）使用 `helm repo add`，因為 Helm 與 OCI-based registries 的互動方式與傳統的 Helm chart repository 不同。

OCI-based registries 和 Helm Chart repository 之間的主要區別：

1. 傳統的 Helm Repositories：
    - 傳統的 Helm repositories 使用 `helm repo add` 命令將 repository URL 註冊到 Helm 中。這允許 Helm 使用簡單的 chart 名稱和版本來搜索、獲取和安裝來自該 repository 的 charts。
    - Repositories 存儲一個 `index.yaml` 檔案，該檔案作為所有 charts 的目錄，Helm 使用它來按名稱獲取 charts。
2. OCI-based registries：
    - OCI-based registries 更類似於 Docker registries（例如，Docker Hub），其中 charts 作為 OCI artifacts。
    - 您不需要索引檔（例如 `index.yaml`）或註冊的步驟（`helm repo add`）。相反，您可以直接使用 `helm pull`、`helm push` 和使用 `oci://` 協議的 `helm install` 命令與 OCI-based registries 進行互動。

使用 OCI-based registries 時，Helm 直接使用 `oci://` 協議與 charts 進行互動，繞過了對 `helm repo add` 和 `index.yaml` 檔案的需求。這更像是與 Docker 映像打交道，而不是傳統的 Helm chart repositories。

### 應用 dependency
讓我們在 `values.yaml` 中添加一個空的 `secrets.passcode`:

{{< collapse openByDefault=true summary="git diff charts/myapi/values.yaml" >}}
```diff
diff --git a/charts/myapi/values.yaml b/charts/myapi/values.yaml
index 288c7d3..a1e50b7 100644
--- a/charts/myapi/values.yaml
+++ b/charts/myapi/values.yaml
@@ -110,3 +110,6 @@ nodeSelector: {}
 tolerations: []
 
 affinity: {}
+
+secrets:
+  passcode: ""
\ No newline at end of file
```
{{< /collapse >}}

然後修改我們的 passcode:
{{< collapse openByDefault=true summary="git diff charts/myapi/templates/secret.yaml" >}}
```diff
diff --git a/charts/myapi/templates/secret.yaml b/charts/myapi/templates/secret.yaml
index 6cd83d6..a81ce6b 100644
--- a/charts/myapi/templates/secret.yaml
+++ b/charts/myapi/templates/secret.yaml
@@ -1,9 +1,12 @@
+{{- $fullname := include "myapi.fullname" . }}
 apiVersion: v1
 kind: Secret
 metadata:
-  name: {{ include "myapi.fullname" . }}
+  name: {{ $fullname }}
   labels:
     {{- include "myapi.labels" . | nindent 4 }}
 type: Opaque
 data:
-  passcode: {{ randAlphaNum  10 | b64enc }}  # Generate a random 10-digit passcode and encode it in base64
+  # Generate secret password or retrieve one if already created.
+  # https://github.com/bitnami/charts/blob/07062ee01382e24b8204b27083ff3e8102110c2f/bitnami/common/templates/_secrets.tpl#L66-L142
+  passcode: {{ include "common.secrets.passwords.manage" (dict "secret" $fullname "key" "passcode" "providedValues" (list "secrets.passcode" ) "length" 10 "strong" false "chartName" "chartName" "context" $) }}
```
{{< /collapse >}}

讓我們來分析一下這些變更：

1. 為 `fullname` 引入變數：
`fullname` 現在被賦值給 `$fullname` 變數，透過 `{{- $fullname := include "myapi.fullname" . }}` 這行。這樣可以在模板中重複使用 `$fullname`，而不必每次都重複寫 `{{ include "myapi.fullname" . }}`，這提高了可讀性，當需要在多個地方使用全名時會更方便。

2. 用管理密碼取代簡單隨機密碼：
簡單的隨機密碼生成 (`randAlphaNum 10`) 被 Bitnami 的 `common.secrets.passwords.manage` 函數所取代。此函數用於生成或檢索密碼，如果密碼已經存在（例如，在升級期間），它會檢索現有的密碼，避免不必要的密碼變更。這增強了密碼管理的安全性和靈活性。該密碼可以在升級中重複使用，確保一致性，避免每次都生成新密碼。更多使用說明可參考其 [GitHub](https://github.com/bitnami/charts/blob/07062ee01382e24b8204b27083ff3e8102110c2f/bitnami/common/templates/_secrets.tpl#L66-L142)。

讓我們卸載並重新安裝它。我們將獲得一個隨機密碼：

{{< collapse openByDefault=true summary="helm upgrade --install myapi-release ./charts/myapi" >}}
```bash
❯ helm uninstall myapi-release
release "myapi-release" uninstalled
❯ helm upgrade --install myapi-release ./charts/myapi
Release "myapi-release" does not exist. Installing it now.
NAME: myapi-release
LAST DEPLOYED: Fri Sep 27 01:35:49 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
❯ kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
x8ZdR8zBsQ%
```
{{< /collapse >}}


我們也可以為它指定一個期望的值。當我們再次安裝時，密碼將保持不變。
{{< collapse openByDefault=true summary="helm upgrade --install myapi-release ./charts/myapi --set secrets.passcode=konosuba" >}}
```bash
❯ helm uninstall myapi-release
release "myapi-release" uninstalled
❯ helm upgrade --install myapi-release ./charts/myapi --set secrets.passcode=konosuba
Release "myapi-release" does not exist. Installing it now.
NAME: myapi-release
LAST DEPLOYED: Fri Sep 27 01:38:49 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
❯ kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
konosuba%
❯ helm upgrade --install myapi-release ./charts/myapi
Release "myapi-release" has been upgraded. Happy Helming!
NAME: myapi-release
LAST DEPLOYED: Fri Sep 27 01:39:06 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=myapi-release" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
❯ kubectl get secret myapi-release -o jsonpath="{.data.passcode}" | base64 --decode
konosuba%
```
{{< /collapse >}}

## 練習五：使用 Chart Releaser Action 自動化 GitHub Pages 上的 Charts
###  設置您的 GitHub Repository
在這個練習中，我們將使用 GitHub Action 將 chart 發佈到 GitHub Pages。首先，您需要一個名為 `k8s-summit-2024` 的新存儲庫。請遵循 [GitHub Pages 快速入門](https://docs.github.com/en/pages/quickstart) 中的第 1、2、3 和 5 步來創建一個名為 k8s-summit-2024 的空 repository：
- 在任何頁面的右上角，選擇 +，然後點擊 **New repository**。
- 輸入 `k8s-summit-2024` 作為 repository 的名稱。
- 選擇 `Public` 作為 repository visibility。
- 添加一些非強制的描述。
- 點擊 **Create repository**。

![create-repo](../../../tech/helm/create-repo.png)


按照 "push an existing repository from the command line" 的指示進行操作. 指令中的 `$USER` 是您的 GitHub 帳戶：

```bash
git remote add origin git@github.com:$USER/k8s-summit-2024.git
git branch -M main
git push -u origin main
```

您還需要一個名為 `gh-pages` 的分支來使其正常運作。我們將使用這個分支來託管我們的 charts。讓我們創建並切換到新分支，然後將其推送到我們的存儲庫。記得切換回您的主分支。

```bash
git checkout -b gh-pages
git push origin gh-pages
git checkout main
```

按照 [GitHub Pages 快速入門](https://docs.github.com/en/pages/quickstart) 中的第 6 步到第 9 步配置您的 gh-pages：

- 在您的 repository 名稱下，點擊 `Settings`。如果您看不到 "Settings" 標籤，請選擇下拉清單，然後點擊 `Settings`。
- 在側邊欄的 "Code and automation" 部分，點擊 `Pages`。
- 在 "Build and deployment" 下，選擇 `Deploy from a branch`。
- 在 "Build and deployment" 下，使用分支下拉菜單選擇 `gh-pages`。

![gh-pages](../../../tech/helm/gh-pages.png)


### 對您的 Helm chart 進行最後的調整

將以下指令複製並貼上至終端機，為您的 Helm chart 新增 `charts/myapi/README.md`。記得將 `ORGNAME` 替換為您的 GitHub 帳號。
```markdown
# Define variables
ALIAS=$USER-k8s-summit-2024
ORGNAME=$USER
CHART_NAME=myapi
REPO_NAME=k8s-summit-2024

# Use sed to replace placeholders and redirect to the chart
sed -e "s/<alias>/$ALIAS/g" \
    -e "s/<orgname>/$ORGNAME/g" \
    -e "s/<chart-name>/$CHART_NAME/g" \
    -e "s/helm-charts/$REPO_NAME/g" << 'EOT' > charts/$CHART_NAME/README.md
## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

    helm repo add <alias> https://<orgname>.github.io/helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve the latest versions of the packages.  You can then run `helm search repo <alias>` to see the charts.

To install the <chart-name> chart:

    helm install <orgname>-<chart-name> <alias>/<chart-name>

To uninstall the chart:

    helm delete <orgname>-<chart-name>
EOT
```

為整個 repository 新增一個 README。
```bash
cat << 'EOT' > README.md
# k8s-summit-2024
A sample helm chart repo created in k8s summit 2024.
EOT
```

將你的名字加入 API 以方便辨識：
{{< collapse openByDefault=true summary="git diff charts/myapi/templates/configmap.yaml" >}}
```diff
diff --git a/charts/myapi/templates/configmap.yaml b/charts/myapi/templates/configmap.yaml
index 0449698..a554438 100644
--- a/charts/myapi/templates/configmap.yaml
+++ b/charts/myapi/templates/configmap.yaml
@@ -13,7 +13,7 @@ data:
 
     @app.get("/")
     def read_root():
-        return {"Hello": f"Your pass code is {os.environ.get('PASSCODE')}"}
+        return {"Hello from mansunkuo": f"Your pass code is {os.environ.get('PASSCODE')}"}
 
     @app.get("/hello/{user}")
     def hello(user: str):
```
{{< /collapse >}}


### 設定 GitHub Actions Workflow

將以下內容複製並貼上至終端機，這會在 `main` 分支的 `.github/workflows/release.yml` 中建立一個 GitHub Actions Workflow 檔案：
```yaml
mkdir -p .github/workflows
cat << 'EOT' > .github/workflows/release.yml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.6.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
EOT
```

您可以在 GitHub Actions Workflow 中找到這個 GitHub Actions 工作流程配置檔案。
這個配置使用了 [@helm/chart-releaser-action](https://github.com/helm/chart-releaser-action) 將您的 GitHub 專案轉變為自我託管的 Helm chart repo。它會在每次推送到 `main` 時檢查您專案中的每個 chart，並在有新的 chart 版本時，建立對應的 GitHub release，該 release 以 chart 版本命名，並將 Helm chart 檔案添加到該 release 中，然後建立或更新 `index.yaml` 檔案，該檔案包含有關這些 release 的 metadata 並託管在 GitHub Pages 上。

當您準備好時，將所有更改添加到提交並推送到遠端的 `main` 分支：
```bash
git add --all
git commit -m lab5
git push origin main
```

稍微等候一下 GitHub Actions Workflow。您將在 `gh-pages` 分支和頁面的根目錄中看到一個 `index.yaml`，其內容大致如下：
```yaml
apiVersion: v1
entries:
  myapi:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2024-10-05T16:02:23.746980517Z"
    dependencies:
    - name: common
      repository: oci://registry-1.docker.io/bitnamicharts
      version: 2.x.x
    description: A Helm chart for Kubernetes
    digest: e4b33c1eb939e05a69a11c20ef81efff5d639a361ee20e2a5372e079bdbb70fe
    name: myapi
    type: application
    urls:
    - https://github.com/mansunkuo/k8s-summit-2024/releases/download/myapi-0.1.0/myapi-0.1.0.tgz
    version: 0.1.0
generated: "2024-10-05T16:02:23.746987961Z"

apiVersion: v1
entries:
  myapi:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2024-10-05T16:02:23.746980517Z"
    dependencies:
    - name: common
      repository: oci://registry-1.docker.io/bitnamicharts
      version: 2.x.x
    description: A Helm chart for Kubernetes
    digest: e4b33c1eb939e05a69a11c20ef81efff5d639a361ee20e2a5372e079bdbb70fe
    name: myapi
    type: application
    urls:
    - https://github.com/mansunkuo/k8s-summit-2024/releases/download/myapi-0.1.0/myapi-0.1.0.tgz
    version: 0.1.0
generated: "2024-10-05T16:02:23.746987961Z"
```

您的 release 也會出現在您的 [Releases](https://github.com/mansunkuo/k8s-summit-2024/releases)。

### 安裝遠端的 Chart
我們在前一步中已經新增了 README 檔案。大部分重要的說明都在 chart 的 README 中可以找到。以下是一些執行結果。

新增 repo:
```bash
❯ helm repo add mansunkuo-k8s-summit-2024 https://mansunkuo.github.io/k8s-summit-2024
"mansunkuo-k8s-summit-2024" has been added to your repositories
```
<br>

列出 chart repositories:
```bash
❯ helm repo list
NAME                            URL                                        
mansunkuo-k8s-summit-2024       https://mansunkuo.github.io/k8s-summit-2024
```
<br>

搜尋 chart:
```bash
❯ helm search repo mansunkuo-k8s-summit-2024
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                
mansunkuo-k8s-summit-2024/myapi 0.1.0           1.16.0          A Helm chart for Kubernetes
```
<br>

安裝 chart:
```bash
❯ helm install mansunkuo-myapi mansunkuo-k8s-summit-2024/myapi
NAME: mansunkuo-myapi
LAST DEPLOYED: Sun Oct  6 02:25:46 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=myapi,app.kubernetes.io/instance=mansunkuo-myapi" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

就這樣，你已經在自己的環境中發布並安裝了一個新的 Helm chart。感謝你為這個美好的世界帶來一個新的 Helm chart。

## 參考資料
- [Chart Releaser Action to Automate GitHub Page Charts](https://helm.sh/docs/howto/chart_releaser_action/)
- [Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart)
- [The Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/)
- [Use OCI-based registries](https://helm.sh/docs/topics/registries/)
- https://helm.sh/docs/chart_template_guide/debugging/