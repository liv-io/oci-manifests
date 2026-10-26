# bitcoin-core

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
    - [bitcoin-core](#bitcoin-core)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `bitcoin-core`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[bitcoin-core](https://github.com/liv-io/oci-containers/tree/main/bitcoin-core)|Container|`31.1-1`|
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
    name: bitcoin-core
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```

- Define bitcoin-core secrets
  ```
  kubectl -n bitcoin-core create secret generic bitcoin-core-secrets --from-literal=RPCAUTH='satoshi:d1eba766ee04c4920d14c63c5706981d$ea702a2c26661fde4096a1542efeb82406ea538ea2487b11d2d4f58d4e96c199'
  ```

### Infra

#### local-path-provisioner

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#local-path-provisioner)

### Apps

#### bitcoin-core

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/bitcoin-core
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/bitcoin-core/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//bitcoin-core?ref=main

  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/bitcoin-core/
  kubectl apply --dry-run=server -k ./apps/bitcoin-core/
  kubectl apply -k ./apps/bitcoin-core/
  ```

- Validate deployment
  ```
  kubectl rollout status -n bitcoin-core deployment/bitcoin-core
  kubectl logs -n bitcoin-core deployment/bitcoin-core --tail=100
  ```

- Get `bitcoin-core` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://bitcoin-core.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment bitcoin-core --replicas=0 -n bitcoin-core
  ```

- Start container
  ```
  kubectl scale deployment bitcoin-core --replicas=1 -n bitcoin-core
  ```

- Inspect container
  ```
  kubectl exec -it -n bitcoin-core deployment/bitcoin-core -- ps -ef
  kubectl exec -it -n bitcoin-core deployment/bitcoin-core -- bitcoind -datadir=/var/local/bitcoin-core/db --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n bitcoin-core
  ```

- Inspect logs
  ```
  kubectl logs -n bitcoin-core deployment/bitcoin-core --tail=100 -f
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

- [bitcoin-core](https://lightning.engineering)
