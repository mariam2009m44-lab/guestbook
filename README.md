# Guestbook — Kubernetes Final Project

Repository for the Guestbook application used in the Skills Network / IBM container and Kubernetes lab.

## Repository layout

- `v1/guestbook/` — Guestbook v1 source, Dockerfile, and Kubernetes Deployment.
- `v2/guestbook/` — Guestbook v2 source and Dockerfile.
- `service.yml` — Service exposing the app on port 3000.
- `hpa.yml` — CPU-based HorizontalPodAutoscaler.
- `deployment-v2.yml` — v2 Deployment manifest for the rolling update.

## Lab registry namespace

The Skills Network terminal showed this IBM Container Registry namespace:

```text
sn-labs-mariam2009m44
```

Use this namespace only in the matching Skills Network lab account.

## Build and push v1

From the Cloud IDE terminal:

```bash
cd /home/project
git clone https://github.com/mariam2009m44-lab/guestbook.git
cd guestbook/v1/guestbook
docker build -t us.icr.io/sn-labs-mariam2009m44/guestbook:v1 .
docker push us.icr.io/sn-labs-mariam2009m44/guestbook:v1
ibmcloud cr images
```

The Docker build must run from `v1/guestbook`, because that directory is the build context and contains the Dockerfile and public assets.

## Deploy v1 to Kubernetes

From `/home/project/guestbook/v1/guestbook`:

```bash
kubectl apply -f deployment.yml
kubectl apply -f ../../service.yml
kubectl apply -f ../../hpa.yml
kubectl get deployment,pods,service,hpa
```

The manifests use the namespace and image tag shown above. Capture the actual output from your lab; never use sample output as evidence.

## Autoscaling check

```bash
kubectl get hpa guestbook
kubectl get hpa guestbook --watch
```

The HPA is configured for a 5% CPU utilization target, with a minimum of 1 and maximum of 10 replicas. Scaling depends on the cluster's metrics API and actual CPU load.

## Build and deploy v2

Build and push v2 from its own build context:

```bash
cd /home/project/guestbook/v2/guestbook
docker build -t us.icr.io/sn-labs-mariam2009m44/guestbook:v2 .
docker push us.icr.io/sn-labs-mariam2009m44/guestbook:v2
```

Apply the v2 manifest. It changes both the image tag and CPU resources (v1: request 2m / limit 5m; v2: request 5m / limit 10m), creating a Deployment revision that can be inspected in rollout history:

```bash
cd /home/project/guestbook
kubectl apply -f deployment-v2.yml
kubectl rollout status deployment/guestbook
kubectl rollout history deployment/guestbook
kubectl rollout history deployment/guestbook --revision=2
```

If the cluster assigns a revision number other than 2, use the revision shown by `kubectl rollout history`; do not assume a fixed number.

## Rollback

Roll back to the previous revision shown in the rollout history. If the initial deployment is revision 1:

```bash
kubectl rollout undo deployment/guestbook --to-revision=1
kubectl rollout status deployment/guestbook
kubectl get rs
```

The ReplicaSet output should be captured only after the rollback command completes. Actual revision numbers and ReplicaSet counts depend on the cluster's rollout history.

## Important notes

- This repository contains source files and manifests; it does not contain proof that the image was built, pushed, or deployed.
- Commands that query the registry or Kubernetes cluster must be run in the active Skills Network Cloud IDE.
- The optional OpenShift Internal Registry lab is a separate deployment path from the Kubernetes final-project assessment. For the optional lab, import the pushed IBM registry image into an OpenShift ImageStream before deploying it from the OpenShift console.
- Save screenshots or copy real terminal output for each assessment item after running the corresponding command.
