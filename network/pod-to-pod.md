# pod-to-pod

Connect to the `busybox-sleep` Pod and try to connect to any of the `app` Pod.

```bash
k exec -it busybox-sleep -- sh
```

## Using the Pod IP address

```bash
/ # wget 10.244.1.50
Connecting to 10.244.1.50 (10.244.1.50:80)
saving to 'index.html'
index.html           100% |************|   615  0:00:00 ETA
'index.html' saved
```

## Using the Pod name

```bash
/ # wget nginx-6bfd886fb5-fvngf
wget: bad address 'nginx-6bfd886fb5-fvngf'
```

In general a Pod has the following DNS resolution:

```bash
pod-ip-address.my-namespace.pod.cluster-domain.example.
```

For example, if a Pod in the `default` namespace has the IP address 172.17.0.3, and the domain name for your cluster is `cluster.local`, then the Pod has a DNS name:

```bash
172-17-0-3.default.pod.cluster.local.
```

Any Pods exposed by a Service have the following DNS resolution available:

```bash
pod-ip-address.service-name.my-namespace.svc.cluster-domain.example.
```

Trying again with the Pod DNS name:

```bash
/ # wget 10-244-1-50.default.pod.cluster.local
Connecting to 10-244-1-50.default.pod.cluster.local (10.244.1.50:80)
saving to 'index.html'
index.html           100% |************|   615  0:00:00 ETA
'index.html' saved
```

Check the [pod-to-service communication](./pod-to-service.md) page for details on how we can use a `Service` for the `nginx` app.

## Acessing a Pod in another namespace