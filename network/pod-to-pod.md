# pod-to-pod communication

- [pod-to-pod communication](#pod-to-pod-communication)
  - [Deploy local environment](#deploy-local-environment)
  - [Using the Pod IP address](#using-the-pod-ip-address)
  - [Using the Pod name](#using-the-pod-name)
  - [Acessing a Pod in a different namespace](#acessing-a-pod-in-a-different-namespace)

## Deploy local environment

```bash
kubectl apply -f ../pods/busybox-pod.yaml
kubectl apply -f ../pods/nginx-deployment.yaml
```

Retrieve the `IP address` of each pod with the following command:

```bash
kubectl get pods -owide
```

Output:

```bash
NAME                     READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
busybox-sleep            1/1     Running   6 (30h ago)   9d    10.244.2.78   minikube   <none>           <none>
nginx-6bfd886fb5-fvngf   1/1     Running   6 (30h ago)   9d    10.244.2.84   minikube   <none>           <none>
nginx-6bfd886fb5-n65xn   1/1     Running   6 (30h ago)   9d    10.244.2.82   minikube   <none>           <none>
nginx-6bfd886fb5-szz7q   1/1     Running   6 (30h ago)   9d    10.244.2.81   minikube   <none>           <none>
```

Connect to the `busybox-sleep` Pod and try to connect to any of the `nginx` Pod as described in the next sections.

```bash
k exec -it busybox-sleep -- sh
```

## Using the Pod IP address

```bash
/ # ping -c1 10.244.2.84
PING 10.244.2.84 (10.244.2.84): 56 data bytes
64 bytes from 10.244.2.84: seq=0 ttl=64 time=0.270 ms

--- 10.244.2.84 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.270/0.270/0.270 ms
```

## Using the Pod name

```bash
/ # ping nginx-6bfd886fb5-fvngf
ping: bad address 'nginx-6bfd886fb5-fvngf'
```

In general a Pod has the following DNS resolution:

```bash
pod-ip-address.my-namespace.pod.cluster-domain.example.
```

For example, if a Pod in the `default` namespace has the IP address 172.17.0.3, and the domain name for your cluster is `cluster.local`, then the Pod has a DNS name:

```bash
172-17-0-3.default.pod.cluster.local.
```

Trying again with the Pod DNS name:

```bash
/ # ping -c1 10-244-2-84.default.pod.cluster.local
PING 10-244-2-84.default.pod.cluster.local (10.244.2.84): 56 data bytes
64 bytes from 10.244.2.84: seq=0 ttl=64 time=0.522 ms

--- 10-244-2-84.default.pod.cluster.local ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.522/0.522/0.522 ms
```

## Acessing a Pod in a different namespace

Create a new `namespace`:

```bash
kubectl create ns example
```

Deploy the `nginx` pods to the `example` namespace:

```bash
kubectl apply -f ../pods/nginx-deployment.yaml --namespace example
```

Retrieve the `IP address` of each pod with the following command:

```bash
kubectl get pods -owide -n example 
```

Output:

```bash                               
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
nginx-6bfd886fb5-4nb4q   1/1     Running   0          11s   10.244.2.86   minikube   <none>           <none>
nginx-6bfd886fb5-9vzjm   1/1     Running   0          11s   10.244.2.87   minikube   <none>           <none>
nginx-6bfd886fb5-kbwrb   1/1     Running   0          11s   10.244.2.85   minikube   <none>           <none>
```

Connect to the `busybox-sleep` Pod and try to connect to any of the `nginx` Pod in the `example` namespace:

```bash
k exec -it busybox-sleep -- sh

/ # wget --spider 10.244.2.87
Connecting to 10.244.2.87 (10.244.2.87:80)
remote file exists

/ # ping -c1 10-244-2-87.default.pod.cluster.local
PING 10-244-2-87.default.pod.cluster.local (10.244.2.87): 56 data bytes
64 bytes from 10.244.2.87: seq=0 ttl=64 time=0.180 ms

--- 10-244-2-87.default.pod.cluster.local ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.180/0.180/0.180 ms
```