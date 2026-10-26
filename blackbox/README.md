# blackbox

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
    - [blackbox](#blackbox)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `blackbox`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[blackbox-exporter](https://quay.io/repository/prometheus/blackbox-exporter)|Container|`0.29.0`|
|[traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)|Helm|`42.1.3`|

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
    name: blackbox
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

#### blackbox

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/blackbox
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/blackbox/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/blackbox/
  kubectl apply --dry-run=server -k ./apps/blackbox/
  kubectl apply -k ./apps/blackbox/
  ```

- Validate deployment
  ```
  kubectl rollout status -n blackbox deployment/blackbox
  kubectl logs -n blackbox deployment/blackbox --tail=100
  ```

- Get `blackbox`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://blackbox.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment blackbox --replicas=0 -n blackbox
  ```

- Start container
  ```
  kubectl scale deployment blackbox --replicas=1 -n blackbox
  ```

- Inspect container
  ```
  kubectl exec -it -n blackbox deployment/blackbox -- ps -ef
  kubectl exec -it -n blackbox deployment/blackbox -- blackbox_exporter --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n blackbox
  ```

- Inspect logs
  ```
  kubectl logs -n blackbox deployment/blackbox --tail=100 -f
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

- [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
