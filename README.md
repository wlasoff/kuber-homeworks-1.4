# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes» - Студент группы FOPS-42 Власов Владимир

# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes»

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться настраивать доступ к приложениям в Kubernetes:
- Внутри кластера через **Service** (ClusterIP, NodePort).
- Снаружи кластера через **Ingress**.

Это задание поможет вам освоить базовые принципы сетевого взаимодействия в Kubernetes — ключевого навыка для работы с кластерами.
На практике Service и Ingress используются для доступа к приложениям, балансировки нагрузки и маршрутизации трафика. Понимание этих механизмов поможет вам упростить управление сервисами в рабочих окружениях и снизит риски ошибок при развёртывании.

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой).
- Установлен `kubectl`.
- Редактор для YAML-файлов (VS Code, Vim и др.).

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) по установке Minikube. 
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)по установке kubectl.
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

## **Задание 1: Настройка Service (ClusterIP и NodePort)**
### **Задача**
Развернуть приложение из двух контейнеров (`nginx` и `multitool`) и обеспечить доступ к ним:
- Внутри кластера через **ClusterIP**.
- Снаружи через **NodePort**.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами:
   - `nginx` (порт `80`).
   - `multitool` (порт `8080`).
   - Количество реплик: `3`.
2. **Создать Service типа ClusterIP**, который:
   - Открывает `nginx` на порту `9001`.
   - Открывает `multitool` на порту `9002`.
3. **Проверить доступность** изнутри кластера:
```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```
4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.
5. **Проверить доступ** с локального компьютера:
```bash
 curl <node-ip>:<node-port>
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-multi-container.yaml`
  - `service-clusterip.yaml`
  - `service-nodeport.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## **Задание 2: Настройка Ingress**
### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
   - `backend` (образ `wbitt/network-multitool`).
2. **Создать Service** для каждого приложения.
3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
   ```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-frontend.yaml`
  - `deployment-backend.yaml`
  - `service-frontend.yaml`
  - `service-backend.yaml`
  - `ingress.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # ПРИМЕР: "multi-container-app"
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ метку из selector.matchLabels
    spec:
      containers:
 - name: # ЗАДАНИЕ: Название первого контейнера
        image: nginx
        ports:
 - containerPort: 80
 - name: multitool
        image: wbitt/network-multitool
        ports:
 - containerPort: 8080
        env:
 - name: HTTP_PORT
          value: "8080" # КЛЮЧЕВОЙ МОМЕНТ: Порт должен совпадать с containerPort
```
### **2. Ingress (для frontend и backend)**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: # ЗАДАНИЕ: Придумайте имя, допустим example-ingress
  annotations:  # ВАЖНО: Эта аннотация нужна для rewrite правил
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
 - http:
      paths:
 - path: /
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя frontend Service
            port:
              number: 80
 - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя backend Service
            port:
              number: 80
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами:
   - `nginx` (порт `80`).
   - `multitool` (порт `8080`).
   - Количество реплик: `3`.
2. **Создать Service типа ClusterIP**, который:
   - Открывает `nginx` на порту `9001`.
   - Открывает `multitool` на порту `9002`.
3. **Проверить доступность** изнутри кластера:
```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```
4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.
5. **Проверить доступ** с локального компьютера:
```bash
 curl <node-ip>:<node-port>
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-multi-container.yaml`
  - `service-clusterip.yaml`
  - `service-nodeport.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## **Задание 2: Настройка Ingress**
### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
   - `backend` (образ `wbitt/network-multitool`).
2. **Создать Service** для каждого приложения.
3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
   ```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-frontend.yaml`
  - `deployment-backend.yaml`
  - `service-frontend.yaml`
  - `service-backend.yaml`
  - `ingress.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

------

---

## Выполнение Задания 1. Настройка Service (ClusterIP и NodePort):
1. Создать Deployment с двумя контейнерами.
```bash
root@ubuntu2204:~/Desktop/4-networks$ nano deployment-multi-container.yml
```
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multi-container-app
  template:
    metadata:
      labels:
        app: multi-container-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.29.8
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            value: "8080"
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f deployment-multi-container.yml
deployment.apps/multi-container-app-deployment created
```

2. Создать Service типа ClusterIP.
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# nano service-clusterip.yml
```
```yml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: multi-container-app
  ports:
    - name: nginx-port
      port: 9001
      targetPort: 80
      protocol: TCP
    - name: multitool-port
      port: 9002
      targetPort: 8080
      protocol: TCP
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f service-clusterip.yml
service/clusterip-service created
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl get all
NAME                                                 READY   STATUS    RESTARTS      AGE
pod/multi-container-app-deployment-b6c9674d9-fwc6b   2/2     Running   0             4m40s
pod/multi-container-app-deployment-b6c9674d9-gkb78   2/2     Running   0             4m40s
pod/multi-container-app-deployment-b6c9674d9-xfvhr   2/2     Running   0             4m40s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/clusterip-service   ClusterIP   10.152.183.180   <none>        9001/TCP,9002/TCP   71s

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/multi-container-app-deployment   3/3     3            3           4m40s

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/multi-container-app-deployment-b6c9674d9   3         3         3       4m40s
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl get ep
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                ENDPOINTS                                                         AGE
clusterip-service   10.1.141.100:8080,10.1.141.98:8080,10.1.141.99:8080 + 3 more...   3m24s
kubernetes          192.168.218.128:16443                                             20d
```

3. **Проверить доступность** изнутри кластера.
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh

# Проверить nginx
/ # curl clusterip-service:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>
<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# Проверить multitool
/ # curl clusterip-service:9002
WBITT Network MultiTool (with NGINX) - multi-container-app-deployment-b6c9674d9-xfvhr - 10.1.141.98 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

# Выходим [Ctrl]+[D]
Session ended, resume using 'kubectl attach test-pod -c test-pod -i -t' command when the pod is running
pod "test-pod" deleted
```

4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# nano service-nodeport.yml
```
```yml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: NodePort
  selector:
    app: multi-container-app
  ports:
    - name: nginx-port
      port: 9001
      targetPort: 80
      protocol: TCP
    - name: multitool-port
      port: 9002
      targetPort: 8080
      protocol: TCP
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f service-nodeport.yml
service/clusterip-service configured
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl get all
NAME                                                 READY   STATUS    RESTARTS      AGE
pod/multi-container-app-deployment-b6c9674d9-fwc6b   2/2     Running   0             19m
pod/multi-container-app-deployment-b6c9674d9-gkb78   2/2     Running   0             19m
pod/multi-container-app-deployment-b6c9674d9-xfvhr   2/2     Running   0             19m

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
service/clusterip-service   NodePort    10.152.183.180   <none>        9001:30747/TCP,9002:32161/TCP   16m
service/kubernetes          ClusterIP   10.152.183.1     <none>        443/TCP                         20d

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/multi-container-app-deployment   3/3     3            3           19m

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/multi-container-app-deployment-b6c9674d9   3         3         3       19m
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl get ep
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                ENDPOINTS                                                         AGE
clusterip-service   10.1.141.100:8080,10.1.141.98:8080,10.1.141.99:8080 + 3 more...   18m
```

5. **Проверить доступ** с локального компьютера:
```bash
C:\Users\wlasoff>curl curl 192.168.218.128:32161                                                                   curl: (6) Could not resolve host: curl                                                                                  WBITT Network MultiTool (with NGINX) - multi-container-app-deployment-b6c9674d9-fwc6b - 10.1.141.99 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool) 
```
```bash
C:\Users\wlasoff>curl 192.168.218.128:30747                                                                        <!DOCTYPE html>                                                                                                         <html>                                                                                                                  <head>                                                                                                                  <title>Welcome to nginx!</title>                                                                                        <style>                                                                                                                 html { color-scheme: light dark; }                                                                                      body { width: 35em; margin: 0 auto;                                                                                     font-family: Tahoma, Verdana, Arial, sans-serif; }                                                                      </style>                                                                                                                </head>                                                                                                                 <body>                                                                                                                  <h1>Welcome to nginx!</h1>                                                                                              <p>If you see this page, nginx is successfully installed and working.                                                   Further configuration is required for the web server, reverse proxy,                                                    API gateway, load balancer, content cache, or other features.</p>                                                                                                                                                                               <p>For online documentation and support please refer to                                                                 <a href="https://nginx.org/">nginx.org</a>.<br/>                                                                        To engage with the community please visit                                                                               <a href="https://community.nginx.org/">community.nginx.org</a>.<br/>                                                    For enterprise grade support, professional services, additional                                                         security features and capabilities please refer to                                                                      <a href="https://f5.com/nginx">f5.com/nginx</a>.</p>                                                                                                                                                                                            <p><em>Thank you for using nginx.</em></p>                                                                              </body>                                                                                                                 </html>  
```

---

## Задание 2. Настройка Ingress

### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
   - `backend` (образ `wbitt/network-multitool`).
2. **Создать Service** для каждого приложения.
3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
```

## Выполнение задания 2. Настройка Ingress:
1. Развернуть два Deployment front и back
## Frontend
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# nano deployment-frontend.yml
```
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.29.8
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "100m"
            limits:
              memory: "128Mi"
              cpu: "200m"
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f deployment-frontend.yml
deployment.apps/frontend-deployment created
```
## Backend
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# # nano deployment-backend.yml
```
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  labels:
    app: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: multitool
          image: wbitt/network-multitool
          ports:
            - containerPort: 8080
          env:
            - name: HTTP_PORT
              value: "8080"
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f deployment-backend.yml
deployment.apps/backend-deployment created
```

2. Создать Service для каждого приложения.
## Frontend
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# # nano service-frontend.yml
```
```yml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
    - name: nginx-port
      port: 80
      targetPort: 80
      protocol: TCP
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f service-frontend.yml
service/frontend-service created
```

## Backend
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# # nano service-backend.yml
```
```yml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - name: multitool-port
      port: 8080
      targetPort: 8080
      protocol: TCP
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f service-backend.yml
service/backend-service created
```

3. Включить Ingress-контроллер
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s enable ingress
Infer repository core for addon ingress
Addon core/ingress is already enabled
```

4. Создать Ingress, который
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.

```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# # nano ingress.yml
```
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 8080
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl apply -f ingress.yml
ingress.networking.k8s.io/example-ingress created
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl get all
NAME                                                 READY   STATUS    RESTARTS      AGE
pod/backend-deployment-6fbcf84f7c-5lz5l              1/1     Running   0             11m
pod/backend-deployment-6fbcf84f7c-wxgt4              1/1     Running   0             11m
pod/backend-deployment-6fbcf84f7c-zbrf7              1/1     Running   0             11m
pod/frontend-deployment-68d9997d78-hzmkx             1/1     Running   0             12m
pod/frontend-deployment-68d9997d78-jq646             1/1     Running   0             12m
pod/frontend-deployment-68d9997d78-t4qx9             1/1     Running   0             12m

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
service/backend-service     ClusterIP   10.152.183.18    <none>        8080/TCP                        29s
service/frontend-service    ClusterIP   10.152.183.157   <none>        80/TCP                          9m4s
service/kubernetes          ClusterIP   10.152.183.1     <none>        443/TCP                         20d

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend-deployment               3/3     3            3           11m
deployment.apps/frontend-deployment              3/3     3            3           12m

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/backend-deployment-6fbcf84f7c              3         3         3       11m
replicaset.apps/frontend-deployment-68d9997d78             3         3         3       12m
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/4-networks# microk8s kubectl describe ingress
Name:             example-ingress
Labels:           <none>
Namespace:        default
Address:          127.0.0.1
Ingress Class:    public
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /      frontend-service:80 (10.1.141.96:80,10.1.141.92:80,10.1.141.94:80)
              /api   backend-service:8080 (10.1.141.95:8080,10.1.141.106:8080,10.1.141.101:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
Events:       <none>
```

5. Проверить доступность
```bash
curl <host>/
curl <host>/api
```
```bash
C:\Users\wlasoff>curl 192.168.218.128                                                                                                            </html> 
```
```bash
curl 192.168.218.128/api
WBITT Network MultiTool (with NGINX) - backend-deployment-6fbcf84f7c-zbrf7 - 10.1.141.106 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
