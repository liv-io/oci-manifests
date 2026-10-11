# alloy

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
    - [alloy](#alloy)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `alloy`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[alloy](https://hub.docker.com/r/grafana/alloy)|Container|`1.20.1`|
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
    name: alloy
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

#### alloy

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/alloy
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/alloy/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/alloy/
  kubectl apply --dry-run=server -k ./apps/alloy/
  kubectl apply -k ./apps/alloy/
  ```

- Validate deployment
  ```
  kubectl rollout status -n alloy deployment/alloy
  kubectl logs -n alloy deployment/alloy --tail=100
  ```

- Get `alloy`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://alloy.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment alloy --replicas=0 -n alloy
  ```

- Start container
  ```
  kubectl scale deployment alloy --replicas=1 -n alloy
  ```

- Inspect container
  ```
  kubectl exec -it -n alloy deployment/alloy -- ps -ef
  kubectl exec -it -n alloy deployment/alloy -- alloy --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n alloy
  ```

- Inspect logs
  ```
  kubectl logs -n alloy deployment/alloy --tail=100 -f
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

- [alloy](https://github.com/grafana/alloy)
