# Kubernetes Network Model

Every `Pod` in a cluster gets its own unique cluster-wide IP address.

Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies):

- pods can communicate with all other pods on any other node without NAT
- agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node

Related Content:

- [Pod-to-Pod communication](./pod-to-pod.md)
  - [Using the Pod IP address](./pod-to-pod.md#using-the-pod-ip-address)
  - [Using the Pod name](./pod-to-pod.md#using-the-pod-name)
  - [Accessing a Pod in another namespace](./pod-to-pod.md#acessing-a-pod-in-another-namespace)
- [Pod-to-Service communication](./pod-to-service.md)
- [Ingress / Ingress Controller](./ingress.md)
- Network Policies (WIP)
