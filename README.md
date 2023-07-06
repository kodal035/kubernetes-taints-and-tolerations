Taints and Tolerations

- Taints allow a node to repel a set of pods.

- Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

- Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

- First of all, we will see that there is no taint in any node.

```bash
kubectl get no
kubectl describe node kube-master | grep -i taint
kubectl describe node kube-worker-1 | grep -i taint
```

- As we see, there is no taint for our nodes.

- We can add taint in the format below.

```bash
kubectl taint nodes node-name key=value:taint-effect
```

- Let's add a taint to the kube-worker-1 using `kubectl taint` command.

```bash
kubectl taint nodes kube-worker-1 clarus=way:NoSchedule
```

- This command places a taint on node kube-worker-1. The taint has key clarus, value way, and taint effect NoSchedule. This means that no pod will be able to schedule onto kube-worker-1 unless it has matching toleration.

- Check that there is a taint for kube-worker-1.

```bash
kubectl describe node kube-worker-1 | grep -i taint
```

- Update the clarus-deploy.yaml as below. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    environment: dev
spec:
  replicas: 15
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- Create the clarus-deploy again.

```bash
kubectl apply -f clarus-deploy.yaml
```

- List the pods and notice that the pods are assigned to the only master node.

```bash
kubectl get po -o wide
```

- We specify toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl taint line above, and thus a pod with either toleration would be able to schedule onto kube-worker-1:

```yaml
tolerations:
- key: "clarus"
  operator: "Equal"
  value: "way"
  effect: "NoSchedule"
```

or

```yaml
tolerations:
- key: "clarus"
  operator: "Exists"
  effect: "NoSchedule"
```

- Update the clarus-deploy.yaml as below.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    environment: dev
spec:
  replicas: 15
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      tolerations:
      - key: "clarus"
        operator: "Exists"
        effect: "NoSchedule"
```

- List the pods.

```bash
kubectl get po -o wide
```

- Run the deployment again.

```bash
kubectl apply -f clarus-deploy.yaml
```

- List the pods and notice that the pods are running at both master node and worker node.

```bash
kubectl get po -o wide
```

- Delete the deployment.

```bash
kubectl delete -f clarus-deploy.yaml
```

> Taints and tolerations are not used to assign a pod to a node, they only restrict nodes from accepting certain tolerations.
> If we want to assign a pod to a specific node, we will use the other concept, Node Affinity.

- Remove taint from the kube-worker-1 node.

```bash
kubectl taint nodes kube-worker-1 clarus=way:NoSchedule-
```