# alertmanager

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
    - [alertmanager](#alertmanager)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `alertmanager`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[alertmanager](https://quay.io/repository/prometheus/alertmanager)|Container|`0.34.1`|
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
    name: alertmanager
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

#### alertmanager

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/alertmanager
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/alertmanager/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/alertmanager/
  kubectl apply --dry-run=server -k ./apps/alertmanager/
  kubectl apply -k ./apps/alertmanager/
  ```

- Validate deployment
  ```
  kubectl rollout status -n alertmanager deployment/alertmanager
  kubectl logs -n alertmanager deployment/alertmanager --tail=100
  ```

- Get `alertmanager`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://alertmanager.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment alertmanager --replicas=0 -n alertmanager
  ```

- Start container
  ```
  kubectl scale deployment alertmanager --replicas=1 -n alertmanager
  ```

- Inspect container
  ```
  kubectl exec -it -n alertmanager deployment/alertmanager -- ps -ef
  kubectl exec -it -n alertmanager deployment/alertmanager -- alertmanager --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n alertmanager
  ```

- Inspect logs
  ```
  kubectl logs -n alertmanager deployment/alertmanager --tail=100 -f
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

- [alertmanager](https://github.com/prometheus/alertmanager)
