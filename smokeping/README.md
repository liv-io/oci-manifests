# smokeping

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
    - [smokeping](#smokeping)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `smokeping`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[smokeping-prober](https://quay.io/repository/superq/smokeping-prober)|Container|`0.12.0`|
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
    name: smokeping
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

#### smokeping

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/smokeping
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/smokeping/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/smokeping/
  kubectl apply --dry-run=server -k ./apps/smokeping/
  kubectl apply -k ./apps/smokeping/
  ```

- Validate deployment
  ```
  kubectl rollout status -n smokeping deployment/smokeping
  kubectl logs -n smokeping deployment/smokeping --tail=100
  ```

- Get `smokeping`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://smokeping.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment smokeping --replicas=0 -n smokeping
  ```

- Start container
  ```
  kubectl scale deployment smokeping --replicas=1 -n smokeping
  ```

- Inspect container
  ```
  kubectl exec -it -n smokeping deployment/smokeping -- ps -ef
  kubectl exec -it -n smokeping deployment/smokeping -- smokeping --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n smokeping
  ```

- Inspect logs
  ```
  kubectl logs -n smokeping deployment/smokeping --tail=100 -f
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

- [smokeping](https://github.com/SuperQ/smokeping_prober)
