# Guestbook (Kubernetes / OpenShift)

This repository contains the two application versions used by the Guestbook deployment lab.

## Source layout

- `v1/guestbook/`: version 1 source, Dockerfile and web assets.
- `v2/guestbook/`: version 2 source, Dockerfile and web assets.
- `deployment.yml`: Kubernetes Deployment template for v1.
- `service.yml`: ClusterIP Service exposing port 3000.
- `hpa.yml`: CPU-based HorizontalPodAutoscaler.

## Before building

The Docker build context must be the matching version directory:

```bash
cd v1/guestbook
docker build -t guestbook:v1 .
```

For v2:

```bash
cd v2/guestbook
docker build -t guestbook:v2 .
```

## IBM Cloud Container Registry

Log in to IBM Cloud and identify a namespace that you own:

```bash
ibmcloud login
ibmcloud cr namespace-list
```

Replace `REPLACE_WITH_YOUR_ICR_NAMESPACE` in `deployment.yml` with that exact namespace. Tag and push the v1 image using the same namespace:

```bash
docker tag guestbook:v1 us.icr.io/YOUR_NAMESPACE/guestbook:v1
docker push us.icr.io/YOUR_NAMESPACE/guestbook:v1
```

Do not submit placeholder text as a real registry result. Capture the actual output of `ibmcloud cr images`.

## Deploy

After setting the image name in `deployment.yml`:

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f hpa.yml
kubectl get deployment,pods,service,hpa
```

The HPA requires a working Kubernetes metrics API and CPU requests on the container. If CPU metrics show `<unknown>`, check the cluster metrics service before testing scaling.

## Version 2 and rollback

Build and push v2 from `v2/guestbook`, update the Deployment image to `guestbook:v2` (using the full registry path), then apply the manifest. Record rollout history and the actual ReplicaSet output before and after rollback.

This repository supplies manifests and source files; terminal output must be captured from the live lab environment.
