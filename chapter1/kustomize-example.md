# Demo di Kustomize

```sh
mkdir -p my-kustomize/base \
         my-kustomize/overlays/dev \
         my-kustomize/overlays/prod && \

cat > my-kustomize/base/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: generic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: registry.access.redhat.com/ubi8/httpd-24:latest
EOF

cat > my-kustomize/base/kustomization.yaml <<'EOF'
namespace: test
resources:
- deployment.yaml
EOF

cat > my-kustomize/overlays/dev/kustomization.yaml <<'EOF'
namespace: dev-environment

namePrefix: dev-
nameSuffix: -v1

bases:
  - ../../base

patches:
- patch: |-
    - op: replace
      path: /spec/template/spec/containers/0/image
      value: registry.access.redhat.com/ubi10/httpd-24:latest
  target:
    kind: Deployment
    name: myapp
EOF

cat > my-kustomize/overlays/prod/kustomization.yaml <<'EOF'
namespace: production
resources:
- ../../base

patches:
  - path: patch.yaml

configMapGenerator:
- name: hello-app-configmap
  literals:
  - msg="Welcome!"
  - enable="true"
EOF

cat > my-kustomize/overlays/prod/patch.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: generic
spec:
  template:
    spec:
      containers:
      - name: myapp
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
EOF
```

## Comandi

```sh
oc kustomize base

oc apply -k base

oc kustomize overlays/dev

oc kustomize overlays/prod
```
