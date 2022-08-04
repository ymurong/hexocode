---
title: Kubernetes for developer
date:  2020-07-15 12:00:00
tags:
- k8s
---
### Kubernetes API Primitives

Kubernetes API primitive, also known as Kubernetes objects, are the basic building blocks of any application running in Kubernetes. Building and managing Kubernetes applications means working with objects. 

Every object has a spec and status:
[kubernetes-objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)
**Spec**: This defines the desired state of the object
**Status**: This is provided by the kubernetes cluster and contains information about the current state of the object

useful commands:
```bash
kubectl api-resources -o name

kubectl get pods -n kube-system

kubectl get pod $object_name

kubectl get nodes

kubectl get nodes $node_name

kubectl get nodes $node_name -o yaml

kubectl describe node $node_name

```

### Configuration

#### ConfigMaps
A ConfigMap is a Kubernetes Object that stores configuration data in a key-value format. This configuration data can then be used to configure software running in a container, by referencing the ConfigMap in the Pod spec.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
   name: my-config-map
data:
   myKey: myValue
   anotherKey: anotherValue
```

Passing ConfigMap data to a container as an environment variable looks like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: my-config-map
          key: myKey
```

It's also possible to pass ConfigMap data to containers, in the form of file using a mounted volume, like so:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-volume-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(cat /etc/config/myKey) && sleep 3600"]
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config-map
```

useful commandes:

```bash
kubectl logs my-configmap-pod

kubectl logs my-configmap-volume-pod

kubectl exec my-configmap-volume-pod -- ls /etc/config

kubectl exec my-configmap-volume-pod -- cat /etc/config/myKey
```

#### Security Context
A security context defines privilege and access control settings for a Pod or Container.

```yaml
spec:
  securityContext:
    runAsUser: 2001
    fsGroup: 3001
```

#### Resources Requirements
```
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### Secrets

```yml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  myKey: myPassword
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-secret-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: myKey
```

### Multi-container pods

* shared network
All listening ports are accessible to other containers in the pod, even if they are not exposed outside the pod

* shared storage volumes
Containers can interact with each other by reading and modifying files in a shared storage volume that is mounted within both containers

* shared process naming space
containers in the same pod cant interact with and signal one another's processes

#### 3 design patterns for multi-container pods
* sidecar pod
enhances or adds functionnality to the main container in some way. For example, a sidecar that syncs files from Git repository to the file system in a web server container
* ambassador pattern uses an amabssador container to accept network traffic and pass it on to the main container. For example, an ambassador that listens on a custom port and forwards traffic to the main container on its hard coded port
* adapter pod
uses an adapter containter to change the output of the main container in some way. An example could be an adapter that formats and decorates log output from the main container


### Observability

#### Probes
Kubernetes probes provide the ability to customize how Kubernetes detects the status of containers, allowing us to build more sophisticated mechanisms for managing container health.
#### Liveness probe
indicates whether the container is running properly, and governs when the cluster will automatically stop or restart the container

```
apiVersion: v1
kind: Pod
metadata:
  name: my-liveness-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    livenessProbe:
      exec:
        command:
        - echo
        - testing
      initialDelaySeconds: 5
      periodSeconds: 5
```

#### Readiness probe
indicates whether the container is ready to service requests, and governs whether requests will be forwarded to the pod

```
apiVersion: v1
kind: Pod
metadata:
  name: my-readiness-pod
spec:
  containers:
  - name: myapp-container
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```









