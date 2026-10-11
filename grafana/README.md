# grafana

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
    - [grafana](#grafana)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `grafana`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[grafana](https://hub.docker.com/grafana/grafana)|Container|`13.2.3`|
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
    name: grafana
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

#### grafana

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/grafana
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/grafana/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/grafana/
  kubectl apply --dry-run=server -k ./apps/grafana/
  kubectl apply -k ./apps/grafana/
  ```

- Validate deployment
  ```
  kubectl rollout status -n grafana deployment/grafana
  kubectl logs -n grafana deployment/grafana --tail=100
  ```

- Get `grafana`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://grafana.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment grafana --replicas=0 -n grafana
  ```

- Start container
  ```
  kubectl scale deployment grafana --replicas=1 -n grafana
  ```

- Inspect container
  ```
  kubectl exec -it -n grafana deployment/grafana -- ps -ef
  kubectl exec -it -n grafana deployment/grafana -- grafana --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n grafana
  ```

- Inspect logs
  ```
  kubectl logs -n grafana deployment/grafana --tail=100 -f
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

- [grafana](https://github.com/grafana/grafana)
