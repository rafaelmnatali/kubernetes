# Kubernetes

Repository with documentation and examples for K8s

## Installing K8s with kubeadm

[This tutorial](./k8s-install/README.md) demonstrates how to provision a Kubernetes cluster using `kubeadm` in a Linux machine.

## Network Model

The [Kubernetes Network Model examples](./network/network.md) you can see here are based on the [Kubernetes Documentation](https://kubernetes.io/docs/concepts/cluster-administration/networking/).

This section will explain:

- [Pod-to-Pod communication](./network/pod-to-pod.md)
  - [Using the Pod IP address](./network/pod-to-pod.md#using-the-pod-ip-address)
  - [Using the Pod name](./network/pod-to-pod.md#using-the-pod-name)
  - [Accessing a Pod in another namespace](./network/pod-to-pod.md#acessing-a-pod-in-another-namespace)
- [Pod-to-Service communication](./network/pod-to-service.md)
- [Ingress / Ingress Controller](./network/ingress.md)
- [Network Policies](./network/network-policy.md)
- [Exercises](./network/exercise.md)

## Contributing

Feel free to contribute by opening issues or pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.
