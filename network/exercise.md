# Network - Exercise

- [Network - Exercise](#network---exercise)
  - [Problem](#problem)
  - [Tasks](#tasks)
  - [Deploy](#deploy)
  - [How would I tackle this issue](#how-would-i-tackle-this-issue)

## Problem

:x: Users are unable to access the `mongodb` database using the MongoDB admin interface.

## Tasks

1. Identify the cause of the `mongo-express` Pods not starting.

   ```bash
   kubectl get pods -w -n network-exercise-frontend     
   NAME                             READY   STATUS    RESTARTS   AGE
   mongo-express-79bd454fbc-p9x4v   0/1     Running   0          4s
   mongo-express-79bd454fbc-p9x4v   0/1     Error     0          35s
   mongo-express-79bd454fbc-p9x4v   0/1     Running   1 (4s ago)   37s
   mongo-express-79bd454fbc-p9x4v   0/1     Error     1 (33s ago)   66s
   mongo-express-79bd454fbc-p9x4v   0/1     CrashLoopBackOff   1 (4s ago)    70s
   ```

   Skills: Troubleshooting, pod-to-pod, and pod-to-service communication

2. Allow only the `mongo-express` to connect to the `mongodb` Pod.

   Skill: Network Policy

3. Enable users to access `mongo-express` from their local computers.

   Skill: Ingress

## Deploy

Deploy the exercise environment with the following command:

```bash
kubectl apply -f ../manifests/exercise-env.yaml
```

Here's a breakdown of what this file does:

- *Namespace*: two namespaces are created: `network-exercise-frontend` and `network-exercise-backend`
- *Deployment and Services*:
  - `mongo-express` in the `network-exercise-frontend`
  - `mongodb` in the `network-exercise-backend`
- *Network Policy*: to control access to the `mongodb` deployment

## How would I tackle this issue

1. check the logs for the `mongo-express` Pod in the `network-exercise-frontend`.
2. create a temporary Pod in the `network-exercise-backend` backend and use the `telnet` command to connect to the `mongodb` Pod directly and using the Service.
3. look for any `NetworkPolicy` that might be restricting communication between the two namespaces.
4. repeat step 2 but now in the `network-exercise-frontend` to confirm that only `mongo-express` Pod can connecto to `mongodb`.
5. verify the Ingress configuration for the users to access `mongo-express` from their local computers

:warning: **Notes:**

- if you are using `minikube` you might need to use this command to allow external communication

```bash
minikube tunnel
minikube service mongo-express-service -n network-exercise-frontend
```

- you can try to access the `mongo-express` from a Pod using the following command:

```bash
curl http://admin:pass@mongo-express-service.network-exercise-frontend:8081
```
