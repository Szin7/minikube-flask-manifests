# Minikube Flask Manifests

This repository contains the Kubernetes and Argo CD configuration for my [Minikube Flask App](https://github.com/Szin7/minikube-flask-app) project.

The application code and GitHub Actions workflow are kept in the other repository. This one holds the deployment configuration that Argo CD reads and applies to Minikube.

## Files

```text
argocd/
  application.yaml
k8s/
  deployment.yaml
  service.yaml
.gitattributes
```

- **`k8s/deployment.yaml`** — runs two Flask replicas using the image published to Docker Hub. It defines the port, health probes and resource settings.
- **`k8s/service.yaml`** — creates an internal ClusterIP Service on port `80`, forwarding traffic to container port `5000`.
- **`argocd/application.yaml`** — tells Argo CD to watch `main`, read the `k8s` directory and deploy to namespace `flask-app`. Automatic sync, prune and self-heal are enabled.

## How it stays up to date

When code is pushed to the [application repository](https://github.com/Szin7/minikube-flask-app), GitHub Actions builds and publishes a Docker image. After the image is pushed, the workflow updates the `image:` field in `k8s/deployment.yaml` and commits that change here as `github-actions[bot]`.

Argo CD detects the change and synchronizes the cluster. The GitHub Actions runner does not require direct access to Minikube.

## Applying the Argo CD application

With Minikube running and Argo CD installed, register the application once from the root of this repository:

```bash
kubectl apply -f argocd/application.yaml
```

Check the application and its Kubernetes resources:

```bash
kubectl get applications -n argocd
kubectl get pods -n flask-app
kubectl get deployment -n flask-app
```

To access the Flask app from the laptop:

```bash
kubectl port-forward -n flask-app svc/minikube-flask-app 8080:80
```

Then open `http://localhost:8080`.

## Deployment check

On **4 November 2026**, the end-to-end test changed the Flask application from `v1` to `v2`.

- App commit: [`5fd5860`](https://github.com/Szin7/minikube-flask-app/commit/5fd5860)
- Automated manifests commit: [`b63336f`](https://github.com/Szin7/minikube-flask-manifests/commit/b63336f)
- Published image at the test: `syjinxz77/minikube-flask-app:5fd5860`
- Argo CD: **Synced** and **Healthy**
- Kubernetes rollout: completed successfully
- Application response: `Hello from minikube-flask-app - version v2`

This is the verified version-change test. Subsequent workflow runs may update the image tag again, including runs triggered by documentation commits.

The [application README](https://github.com/Szin7/minikube-flask-app#readme) contains the full pipeline explanation and screenshots.
