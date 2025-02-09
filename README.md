# Backstage QShift Showcase

The backstage QShift application has been designed to showcase QShift (Quarkus on OpenShift) and integrate the following plugins:
- [Kubernetes plugin](https://backstage.io/docs/features/kubernetes/installation) - [pr/commit](https://github.com/q-shift/backstage-playground/pull/5)
- [Quarkus plugin](https://github.com/q-shift/backstage-plugins)
- ArgoCD [front](https://github.com/RoadieHQ/roadie-backstage-plugins/tree/main/plugins/frontend/backstage-plugin-argo-cd) & [backend](https://github.com/RoadieHQ/roadie-backstage-plugins/tree/main/plugins/scaffolder-actions/scaffolder-backend-argocd) - [pr/commit](https://github.com/q-shift/backstage-playground/pull/7)
- [Tekton Plugin](https://github.com/janus-idp/backstage-plugins/tree/main/plugins/tekton) - [pr/commit]()

**Note**: It has been developed using backstage version: 1.21.0

## Prerequisites

- [Node.js](https://nodejs.org/en) (18 or 20)
- [nvm](https://github.com/nvm-sh/nvm), npm and [yarn](https://classic.yarnpkg.com/lang/en/docs/install/#mac-stable) installed
- Read this blog post: https://medium.com/@chrisschneider/build-a-developer-portal-with-backstage-on-openshift-d2a97aca91ee

## Instructions

### Locally
To use this project, git clone it and run the following command:

```sh
yarn install
yarn start --config ../../app-config.qshift.yaml
yarn start-backend --config ../../app-config.qshift.yaml
```

**Warning**: If you use node 20, then export the following env var `export NODE_OPTIONS=--no-node-snapshot` as documented [here](https://backstage.io/docs/getting-started/configuration/#create-a-new-component-using-a-software-template).
### On OCP

TODO: To be reviewed please !

First, log on to the ocp cluster and verify that OpenShift GitOps operator has been installed

Create the following configMap containing the backstage configuration file `app-config.local.yaml` under the project/namespace `openshift-gitops`
```bash
NAMESPACE=openshift-gitops
kubectl create configmap my-app-config -n $NAMESPACE \
  --from-file=app-config.local.yaml=$(pwd)/manifest/app-config.local.yaml \
  -o yaml --dry-run=client | kubectl apply -n $NAMESPACE -f -
```

Deploy using ArgoCD the Backstage Helm chart:
```bash
kubectl apply -f manifest/argocd.yaml
```

## Additional resources

ArgoCD: https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml

DOMAIN_NAME=127.0.0.1.nip.io
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.$DOMAIN_NAME
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
EOF
```
