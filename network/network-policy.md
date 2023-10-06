# Network Policies

- [Network Policies](#network-policies)
  - [Definition](#definition)
  - [Restricting Namespace Access](#restricting-namespace-access)

## Definition

If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), then you might consider using [Kubernetes NetworkPolicies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) for particular applications in your cluster.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:

1. Other pods that are allowed (exception: a pod cannot block access to itself)
2. Namespaces that are allowed
3. IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

## Restricting Namespace Access

In this example, we want to restrict access to NGINX from Pods in the `external` namespace.

First, create the namespaces:

```bash
kubectl create namespace external
```

Second, deploy the NGINX:

```bash
kubectl apply -f ../manifests/nginx-deployment.yaml 
```

Use the [nicolaka/netshoot](https://hub.docker.com/r/nicolaka/netshoot) image to connect to the NGINX.

```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```

Then:

```bash
tmp-shell:~# curl nginx-service
Welcome to the home page of host nginx-5c7575f557-q66bx!
```

Switch the context to the `external` namespace and repeat the same test with the `nicolaka/netshoot` image:

```bash
tmp-shell:~# curl nginx-service.default
Welcome to the home page of host nginx-5c7575f557-q66bx!
```

:speech_balloon: include the `default` after the Service name because the Service was created in a different namespace.

At first, the connection was successful. Now, apply the `NetworkPolicy`:

```bash
kubectl apply -f ../manifests/np-allow-default.yaml
```

and repeat the test:

```bash
tmp-shell:~# curl nginx-service.default
curl: (28) Failed to connect to nginx-service.default port 80 after 131139 ms: Couldn't connect to server
```

This time it fails because our `NetworkPolicy` only allows `Ingress` communication from the `default` namespace:

```yaml
...
policyTypes:
  - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: default
```
