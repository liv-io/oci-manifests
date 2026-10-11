# btcd

## Index

- [About](#about)
- [Dependencies](#dependencies)
  - [Resources](#resources)
- [Deployment](#deployment)
  - [Bootstrap](#bootstrap)
    - [namespaces](#namespaces)
  - [Infra](#infra)
    - [local-path-provisioner](#local-path-provisioner)
    - [traefik](#traefik)
  - [Apps](#apps)
    - [btcd](#btcd)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `btcd`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[btcd](https://github.com/liv-io/oci-containers/tree/main/btcd)|Container|`0.26.2-2`|
|[local-path-provisioner](https://github.com/rancher/local-path-provisioner)|Kustomize|`0.0.37`|
|[traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)|Helm|`41.7.1`|

## Deployment

- Create GitOps repository
  ```
  git init -b main example-env
  cd ./example-env/
  ```

- Create GitOps directory structure
  ```
  install --directory --mode=0755 ./bootstrap ./apps ./infra
  ```

### Bootstrap

#### namespaces

- Create namespaces file
  ```
  cat <<EOF > ./bootstrap/namespaces.yaml
  apiVersion: v1
  kind: Namespace

  metadata:
    name: btcd
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```

- Define btcd secrets
  ```
  kubectl create secret generic btcd-secrets --namespace=btcd --from-literal=RPCPASS='U-JL.FNdKEU9Vkc-g9AjLk9rrAH.svev'
  ```

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

### Apps

#### btcd

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/btcd
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/btcd/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//btcd?ref=main

  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/btcd/
  kubectl apply --dry-run=server -k ./apps/btcd/
  kubectl apply -k ./apps/btcd/
  ```

- Validate deployment
  ```
  kubectl rollout status -n btcd deployment/btcd
  kubectl logs -n btcd deployment/btcd --tail=100
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment btcd --replicas=0 -n btcd
  ```

- Start container
  ```
  kubectl scale deployment btcd --replicas=1 -n btcd
  ```

- Inspect container
  ```
  kubectl exec -it -n btcd deployment/btcd -- ps -ef
  kubectl exec -it -n btcd deployment/btcd -- btcd --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n btcd
  ```

- Inspect logs
  ```
  kubectl logs -n btcd deployment/btcd --tail=100 -f
  ```

- Get image versions
  ```
  kubectl get nodes -o json | jq -r '.items[].status.images[].names[]' | sort -u
  ```

## License

See `LICENSE` file for more information.

## Credits

See `CREDITS.md` file for more information.

## Appendix

- [btcd](https://github.com/btcsuite/btcd)
