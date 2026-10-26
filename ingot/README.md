# ingot

## Index

- [About](#about)
- [Dependencies](#dependencies)
  - [Resources](#resources)
- [Deployment](#deployment)
  - [Bootstrap](#bootstrap)
    - [namespaces](#namespaces)
  - [Infra](#infra)
    - [traefik](#traefik)
  - [Apps](#apps)
    - [ingot](#ingot)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `ingot`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[ingot](https://github.com/liv-io/oci-containers/tree/main/ingot)|Container|`0.1.0`|
|[traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)|Helm|`41.5.0`|

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
    name: ingot
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```
### Infra

#### traefik

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#traefik)

### Apps

#### ingot

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/ingot
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/ingot/kustomization.yaml
  EOF
  ```

- Validate deployment
  ```
  kubectl rollout status -n ingot deployment/ingot
  kubectl logs -n ingot deployment/ingot --tail=100
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment ingot --replicas=0 -n ingot
  ```

- Start container
  ```
  kubectl scale deployment ingot --replicas=1 -n ingot
  ```

- Inspect container
  ```
  kubectl exec -it -n ingot deployment/ingot -- ps -ef
  kubectl exec -it -n ingot deployment/ingot -- ingot --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n ingot
  ```

- Inspect logs
  ```
  kubectl logs -n ingot deployment/ingot --tail=100 -f
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

- [ingot](https://github.com/liv-io/oci-containers/tree/main/ingot)
