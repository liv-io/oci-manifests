# loki

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
    - [loki](#loki)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `loki`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[loki](https://hub.docker.com/grafana/loki)|Container|`3.7.8`|
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
    name: loki
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```
### Infra

#### local-path-provisioner

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#local-path-provisioner)

#### traefik

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#traefik)

### Apps

#### loki

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/loki
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/loki/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/loki/
  kubectl apply --dry-run=server -k ./apps/loki/
  kubectl apply -k ./apps/loki/
  ```

- Validate deployment
  ```
  kubectl rollout status -n loki deployment/loki
  kubectl logs -n loki deployment/loki --tail=100
  ```

- Get `loki`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://loki.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment loki --replicas=0 -n loki
  ```

- Start container
  ```
  kubectl scale deployment loki --replicas=1 -n loki
  ```

- Inspect container
  ```
  kubectl exec -it -n loki deployment/loki -- ps -ef
  kubectl exec -it -n loki deployment/loki -- loki --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n loki
  ```

- Inspect logs
  ```
  kubectl logs -n loki deployment/loki --tail=100 -f
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

- [loki](https://github.com/grafana/loki)
