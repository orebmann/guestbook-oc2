# Guestbook OC

OpenShift-compatible Guestbook application deployable via ArgoCD.

## What's inside

```
guestbook-oc/
├── chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── _helpers.tpl
│       ├── serviceaccount.yaml   # dedicated SA: guestbook
│       ├── configmap-html.yaml   # static HTML served by httpd
│       ├── deployment.yaml       # non-root, port 8080
│       ├── service.yaml          # ClusterIP on 8080
│       └── route.yaml            # OpenShift Route (edge TLS)
└── argocd-application.yaml       # ArgoCD Application CR
```

## Image

`registry.access.redhat.com/ubi8/httpd-24`

- Runs as **non-root** out of the box — no SCC changes needed
- Listens on **port 8080** (not 80)
- Logs to stdout/stderr — container-native
- Free to pull from Red Hat registry

## Deploy via ArgoCD

1. Push this repo to Git.
2. Edit `argocd-application.yaml` — set `repoURL` and `destination.namespace`.
3. Apply the Application CR:

```bash
oc apply -f argocd-application.yaml
```

ArgoCD will create the namespace, ServiceAccount, ConfigMap, Deployment, Service and Route automatically.

## Deploy manually with Helm (for testing)

```bash
helm install guestbook ./chart -n guestbook --create-namespace
```

## Access the app

```bash
oc get route -n guestbook
```

Open the printed hostname in your browser.

## Why no SCC changes?

`ubi8/httpd-24` is built specifically for OpenShift:
- Runs as UID **1001** (non-root, non-privileged)
- Listens on **8080** — no privileged port
- Writable directories are pre-configured for arbitrary UIDs

No `anyuid` SCC, no `privileged` SCC required.
