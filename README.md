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


## License

Distributed under the Simplified BSD License.

See `LICENSE` file for more information.

## Credits

See `CREDITS.md` file for more information.

## Appendix
