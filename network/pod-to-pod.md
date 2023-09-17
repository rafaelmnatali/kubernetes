# Pod to Pod communication

- [Pod to Pod communication](#pod-to-pod-communication)
  - [Deploy local environment](#deploy-local-environment)
  - [Using the Pod IP address](#using-the-pod-ip-address)
  - [Using the Pod name](#using-the-pod-name)
  - [Accessing a Pod in a different namespace](#accessing-a-pod-in-a-different-namespace)

## Deploy local environment

```bash
kubectl apply -f ../pods/nginx-deployment.yaml
```

Retrieve the detailed information of each pod with the following command:

```bash
kubectl get pods -owide
```

Output:

```bash
NAME                     READY   STATUS    RESTARTS      AGE   IP            NODE       NOMINATED NODE   READINESS GATES
nginx-6bfd886fb5-fvngf   1/1     Running   6 (30h ago)   9d    10.244.2.84   minikube   <none>           <none>
nginx-6bfd886fb5-n65xn   1/1     Running   6 (30h ago)   9d    10.244.2.82   minikube   <none>           <none>
nginx-6bfd886fb5-szz7q   1/1     Running   6 (30h ago)   9d    10.244.2.81   minikube   <none>           <none>
```

Use the [nicolaka/netshoot](https://hub.docker.com/r/nicolaka/netshoot) image to execute network validations.

```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```

## Using the Pod IP address

First, let's send an [ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) request to one of the pods using the [ping](https://man.cx/ping(1)) command:

```bash
tmp-shell:~# ping -c1 10.244.2.84
PING 10.244.2.84 (10.244.2.84) 56(84) bytes of data.
64 bytes from 10.244.2.84: icmp_seq=1 ttl=64 time=0.544 ms

--- 10.244.2.84 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.544/0.544/0.544/0.000 ms
```

We successfully sent and received one request.

As [NGINX](https://www.nginx.com) is a [web server](https://en.wikipedia.org/wiki/Web_server), we can use the [curl](https://everything.curl.dev/project) command to send an [HTTP](https://everything.curl.dev/protocols/curl#http) request.

```bash
tmp-shell:~# curl 10.244.2.84
Welcome to the home page of host nginx-6bfd886fb5-fvngf!
```

```bash
tmp-shell:~# curl 10.244.2.82
Welcome to the home page of host nginx-6bfd886fb5-n65xn!
```

We can see that both requests were successful. Each request connects to a different Pod and returns the web server's home page.

## Using the Pod name

If we try `ping` the Pod using their assigned `name`, we receive an error:

```bash
tmp-shell:~# ping -c1 nginx-6bfd886fb5-fvngf
ping: nginx-6bfd886fb5-fvngf: Name does not resolve
```

That's why, in general, a Pod has the following [DNS resolution](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pods):

```bash
pod-ip-address.my-namespace.pod.cluster-domain.example.
```

This time, use `ping` and `curl` with the Pod DNS name:

```bash
tmp-shell:~# ping -c1 10-244-2-84.default.pod.cluster.local
PING 10-244-2-84.default.pod.cluster.local (10.244.2.84) 56(84) bytes of data.
64 bytes from 10.244.2.84 (10.244.2.84): icmp_seq=1 ttl=64 time=0.169 ms

--- 10-244-2-84.default.pod.cluster.local ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.169/0.169/0.169/0.000 ms
```

```bash
tmp-shell:~# curl 10-244-2-84.default.pod.cluster.local
Welcome to the home page of host nginx-6bfd886fb5-fvngf!
```

Both commands return the expected results.

## Accessing a Pod in a different namespace

This type of request will follow the same principles we saw in the first two sections.

1. Create a new `namespace`:

    ```bash
    kubectl create ns example
    ```

2. Deploy the `nginx` pods to the `example` namespace:

    ```bash
    kubectl apply -f ../pods/nginx-deployment.yaml --namespace example
    ```

3. Retrieve the detailed information of each pod with the following command:

    ```bash
    kubectl get pods -owide -n example 
    ```

    Output:

    ```bash
    NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
    nginx-6bfd886fb5-mfwtd   1/1     Running   0          7s    10.244.2.92   minikube   <none>           <none>
    nginx-6bfd886fb5-mvppc   1/1     Running   0          7s    10.244.2.91   minikube   <none>           <none>
    nginx-6bfd886fb5-pp964   1/1     Running   0          7s    10.244.2.90   minikube   <none>           <none>
    ```

Use the [nicolaka/netshoot](https://hub.docker.com/r/nicolaka/netshoot) image to execute network validations.

```bash
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```

Use `ping` and `curl` to the IP and DNS name of one of the Pods in the `example` namespace:

```bash
tmp-shell:~# ping -c1 10.244.2.90
PING 10.244.2.90 (10.244.2.90) 56(84) bytes of data.
64 bytes from 10.244.2.90: icmp_seq=1 ttl=64 time=0.557 ms

--- 10.244.2.90 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.557/0.557/0.557/0.000 ms
```

```bash
tmp-shell:~# curl 10-244-2-90.example.pod.cluster.local
Welcome to the home page of host nginx-6bfd886fb5-pp964!
```

Again, both commands return the expected results.
