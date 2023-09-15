# Kubernetes Network Model

Every `Pod` in a cluster gets its own unique cluster-wide IP address.

Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies):

- pods can communicate with all other pods on any other node without NAT
- agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node

```bash
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
busybox-sleep            1/1     Running   0          32s   10.244.1.51   minikube   <none>           <none>
nginx-6bfd886fb5-fvngf   1/1     Running   0          89s   10.244.1.50   minikube   <none>           <none>
nginx-6bfd886fb5-n65xn   1/1     Running   0          89s   10.244.1.49   minikube   <none>           <none>
nginx-6bfd886fb5-szz7q   1/1     Running   0          89s   10.244.1.48   minikube   <none>           <none>
```

Related Content:
- [pod-to-pod communication](./pod-to-pod.md)
  - Using the Pod IP address
  - Using the Pod name
  - Acessing a Pod in another namespace
- [pod-to-service communication](./pod-to-service.md)
