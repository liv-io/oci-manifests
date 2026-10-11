# blockbook

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
    - [blockbook](#blockbook)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `blockbook`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[blockbook](https://github.com/liv-io/oci-containers/tree/main/blockbook)|Container|`0.6.0-1`|
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
    name: blockbook
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```

- Define blockbook secrets
  ```
  kubectl create secret generic blockbook-secrets --namespace=blockbook --from-literal=RPCPASS='U-JL.FNdKEU9Vkc-g9AjLk9rrAH.svev'
  ```

### Infra

#### local-path-provisioner

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#local-path-provisioner)

#### traefik

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#traefik)

### Apps

#### blockbook

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/blockbook
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/blockbook/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//blockbook?ref=main

  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/blockbook/
  kubectl apply --dry-run=server -k ./apps/blockbook/
  kubectl apply -k ./apps/blockbook/
  ```

- Validate deployment
  ```
  kubectl rollout status -n blockbook deployment/blockbook
  kubectl logs -n blockbook deployment/blockbook --tail=100
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment blockbook --replicas=0 -n blockbook
  ```

- Start container
  ```
  kubectl scale deployment blockbook --replicas=1 -n blockbook
  ```

- Inspect container
  ```
  kubectl exec -it -n blockbook deployment/blockbook -- ps -ef
  kubectl exec -it -n blockbook deployment/blockbook -- blockbook --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n blockbook
  ```

- Inspect logs
  ```
  kubectl logs -n blockbook deployment/blockbook --tail=100 -f
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

- [blockbook](https://github.com/trezor/blockbook)
