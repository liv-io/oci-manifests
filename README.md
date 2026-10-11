# oci-manifests

## Index

- [About](#about)
- [Features](#features)
- [Deployment](#deployment)
  - [Infra](#infra)
    - [local-path-provisioner](#local-path-provisioner)
    - [traefik](#traefik)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

`oci-manifests` is a collection of well curated OCI manifests.

## Features

- Manifests follow the secure-by-default principle
- Manifests are mostly self-contained and dependencies avoided
- Parameters are documented with examples

### Infra

#### local-path-provisioner

> [!IMPORTANT]
> Deploy `local-path-provisioner` if not included in your Kubernetes distribution

- Create manifest directory
  ```
  install --directory --mode=0755 ./infra/local-path-provisioner
  ```

- Populate manifest
  ```
  cat <<EOF > ./infra/local-path-provisioner/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  namespace: local-path-storage

  resources:
    - https://github.com/rancher/local-path-provisioner/deploy?ref=master

  images:
    - name: rancher/local-path-provisioner
      newTag: v0.0.37

  patches:
    - target:
        kind: StorageClass
        name: local-path
      patch: |-
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: local-path
          annotations:
            storageclass.kubernetes.io/is-default-class: "true"
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./infra/local-path-provisioner/
  kubectl apply --dry-run=server -k ./infra/local-path-provisioner/
  kubectl apply -k ./infra/local-path-provisioner/
  ```

- Validate deployment
  ```
  kubectl rollout status -n local-path-storage deployment/local-path-provisioner
  kubectl get all -n local-path-storage -l app=local-path-provisioner
  ```

#### traefik

- Create manifest directory
  ```
  install --directory --mode=0755 ./infra/traefik
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./infra/traefik/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - https://raw.githubusercontent.com/traefik/traefik/refs/tags/v3.7.14/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
    - https://raw.githubusercontent.com/traefik/traefik/v3.7.14/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
    - git@github.com:liv-io/oci-manifests.git//traefik?ref=main

  configMapGenerator:
    - name: internal-ca-cert
      namespace: kube-system
      behavior: replace
      files:
        - ca1.liv.io.crt

  patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: traefik
          namespace: kube-system
        spec:
          template:
            spec:
              containers:
                - name: traefik
                  env:
                    - name: LEGO_CA_CERTIFICATES
                      value: /etc/traefik/ca/ca1.liv.io.crt

    - target:
        kind: Deployment
        name: traefik
        namespace: kube-system
      patch: |-
        - op: add
          path: /spec/template/spec/containers/0/args/-
          value: --certificatesresolvers.myresolver.acme.caserver=https://ca.liv.io/acme/acme/directory
        - op: add
          path: /spec/template/spec/containers/0/args/-
          value: --certificatesresolvers.myresolver.acme.email=admin@liv.io
  EOF
  ```

- Render and apply manifest
  ```
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply --dry-run=client -f -
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply --dry-run=server -f -
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply -f -
  ```

- Validate deployment
  ```
  kubectl rollout status -n kube-system deployment/traefik
  kubectl get all -n kube-system -l app.kubernetes.io/name=traefik
  ```

## License

Distributed under the Simplified BSD License.

See `LICENSE` file for more information.

## Credits

See `CREDITS.md` file for more information.

## Appendix
