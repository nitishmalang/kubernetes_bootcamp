# Kubernetes Pods

1. Get pods: `kubectl get pods`
2. Get namespaces: `kubectl get ns`
3. Create a new namespace called bootcamp: `kubectl create ns bootcamp`
4. Again get pods: `kubectl get pods`
5. Describe a specific pod: `kubectl describe pod <pod_name>`
6. Create a new pod named abc with nginx image: `kubectl create pod abc --image=nginx`
7. Again get pods to check the newly created pod: `kubectl get pods`
8. If the new pod is running, describe it: `kubectl describe pods <pod_name>`
9. To get more information about pods: `kubectl get pods -owide`

# Pods in declarative way

vi first.yaml 

inside that nano write 

apiVersion: v1
kind: Pod
metadata:
name: example-pod
labels:
purpose: demonstrate-args
spec:
containers:

- name: example-container
image: ubuntu
command: ["/bin/echo", "Hello"]
args: ["Welcome", "to", "Kubesimplify"]

cat first.yaml

kubectl create -f first.yaml

kubectl get pods

kubectl describe pods

# Init Containers

**init container**

```
apiVersion: v1
kind: Pod
metadata:
  name: bootcamp-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  initContainers:
  - name: bootcamp-init
    image: busybox
    command: ['sh', '-c', 'wget -O /usr/share/data/index.html http://kubesimplify.com']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/data
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

```

**multiple init container**

```
apiVersion: v1
kind: Pod
metadata:
  name: init-demo-2
spec:
  initContainers:
  - name: check-db-service
    image: busybox
    command: ['sh', '-c', 'until nslookup db.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
  - name: check-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice.default.svc.cluster.local; do echo waiting for db service; sleep 2; done;']
  containers:
  - name: main-container
    image: busybox
    command: ['sleep', '3600']

```

```
---
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  selector:
    app: demo1
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: demo2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

```

**Multi container pod**

`apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  initContainers:
  - name: meminfo-container
    image: alpine
    restartPolicy: Always
    command: ['sh', '-c', 'sleep 5; while true; do cat /proc/meminfo > /usr/share/data/index.html; sleep 10; done;']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/data
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html`

#
