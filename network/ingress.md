# Ingress / Ingress Controller

- [Ingress / Ingress Controller](#ingress--ingress-controller)
  - [What is Ingress?](#what-is-ingress)
  - [What is an Ingress Controller?](#what-is-an-ingress-controller)
  - [Set up Ingress on Minikube with the NGINX Ingress Controller](#set-up-ingress-on-minikube-with-the-nginx-ingress-controller)

## What is Ingress?

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to [services](https://kubernetes.io/docs/concepts/services-networking/service/) within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

## What is an Ingress Controller?

An [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers) is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

You may need to deploy an Ingress controller such as [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/).

## Set up Ingress on Minikube with the NGINX Ingress Controller

1. To enable the NGINX Ingress controller, run the following command:

    ```bash
    minikube addons enable ingress
    ```

2. Verify that the NGINX Ingress controller is running

    ```bash
    kubectl get pods -n ingress-nginx
    ```

    The output is similar to:

    ```bash
    NAME                                       READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-6hbld       0/1     Completed   0          41s
    ingress-nginx-admission-patch-bnrsb        0/1     Completed   1          41s
    ingress-nginx-controller-77669ff58-x55s2   1/1     Running     0          41s
    ```

3. Create a Deployment using the following command:

    ```bash
    kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
    ```

4. Expose the Deployment:

    ```bash
    kubectl expose deployment web --type=NodePort --port=8080
    ```

5. Verify the Service is created and is available on a node port:

    ```bash
    kubectl get service web
    ```

6. Create the Ingress object by running the following command:

    ```bash
    kubectl apply -f ../manifests/example-ingress.yaml
    ```

7. Verify the IP address is set:

    ```bash
    kubectl get ingress
    ```

8. Verify that the Ingress controller is directing traffic:

    ```bash
    curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
    ```

    :warning: if you can't communicate with the `minikube ip`, open the `minikube tunnel` in another terminal and use `127.0.0.1` instead of the `$( minikube ip )`.

    You should see:

    ```bash
    Hello, world!
    Version: 1.0.0
    Hostname: web-68487bc957-57rb7
    ```
