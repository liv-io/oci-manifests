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

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#local-path-provisioner)

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
