# XOps Task 6 – App Repository

A simple NGINX-hosted static site deployed via GitOps with GitHub Actions and Argo CD into a `kind` Kubernetes cluster.

---

##  Overview

This repo contains the **application code** for XOps Microchallenge #6:
- `index.html`: A styled HTML page.
- `Dockerfile`: Builds the site using NGINX.
- `.github/workflows/task6-cicd.yaml`: CI/CD pipeline that:
  1. Builds the Docker image.
  2. Pushes it to Docker Hub (`<DOCKERHUB_USERNAME>/xops-app:<commit-sha>`).
  3. Updates the image tag in the Infra repo (`xops-infra`) Kubernetes manifest.

Once manifests are updated, **Argo CD detects the change** and syncs them to deploy the app in your local kind cluster.

---

##  Quickstart (Local Workflow)

1. **Edit your UI**  
   Modify `index.html` to update content or style.

2. **Push changes**  
   ```bash
   git add index.html
   git commit -m "Update UI styling"
   git push origin main
   ```

3. **Pipeline runs automatically**  
   - GitHub Actions builds and pushes a new Docker image.
   - CI updates `deployment.yaml` in the Infra repo.
   - Argo CD auto-syncs the changes into Kubernetes.

4. **View the updated site**  
   Forward the service to a local port:
   ```bash
   kubectl port-forward svc/xops-service 9090:80 -n default
   ```
   Open [http://localhost:9090](http://localhost:9090).

---

##  Project Structure

```
├── index.html
├── Dockerfile
├── .github/
│   └── workflows/
│       └── task6-cicd.yaml
└── README.md
```

- **`Dockerfile`**: Uses `nginx:alpine`, serves `index.html`.
- **CI Workflow**:
  - Logs into Docker Hub (`teujksdf`).
  - Builds image tagged by Git commit SHA.
  - Pushes image.
  - Updates Infra repo manifest.

---

##  Required Setup

### On GitHub (App repo)
Add these secrets under **Settings → Secrets and variables → Actions**:

| Name              | Value                                     |
|------------------|-------------------------------------------|
| `DOCKERHUB_USERNAME` | DcokerUsername                            |
| `DOCKERHUB_TOKEN`    | Docker Hub personal access token      |
| `PAT_TOKEN`          | GitHub token with repo/write access for Infra repo |

### Infra Repo (`xops-infra`)
Structure:
```
k8s/
├── deployment.yaml    # references image tag to be updated by CI
└── service.yaml       # exposes the app via NodePort
```
Argo CD watches this repository to deploy changes.

---

##  Project Flow Summary

| Step | Description |
|------|-------------|
| 1. Edit Code | Modify `index.html` as needed. |
| 2. Push Code | `git push` triggers CI. |
| 3. CI Pipeline | Builds/pushes image + updates Infra repo. |
| 4. Infra Update | New image tag committed to Infra repo. |
| 5. Argo CD Sync | Detects change, applies manifests to cluster. |
| 6. Access App | Use port-forward to view updated site. |

---

##  Troubleshooting

- **Site not loading**: Ensure `kubectl port-forward` to access service (use an available port like `9090`).
- **Pipeline fails**: Check that GitHub secrets are correctly set, particularly `PAT_TOKEN` for infra repo updates.
- **App not updated**: Confirm that Infra repo's `deployment.yaml` has been updated with the new image tag.
- **Argo CD reports OutOfSync**: Refresh or re-sync manually via UI or CLI.


