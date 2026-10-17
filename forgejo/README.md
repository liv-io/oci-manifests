# forgejo

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
    - [forgejo](#forgejo)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `forgejo`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[forgejo](https://github.com/liv-io/oci-containers/tree/main/forgejo)|Container|`17.0.0-1`|
|[traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)|Helm|`42.0.0`|

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
    name: forgejo
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

#### forgejo

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/forgejo
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/forgejo/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/forgejo/
  kubectl apply --dry-run=server -k ./apps/forgejo/
  kubectl apply -k ./apps/forgejo/
  ```

- Validate deployment
  ```
  kubectl rollout status -n forgejo deployment/forgejo
  kubectl logs -n forgejo deployment/forgejo --tail=100
  ```

- Get `forgejo` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://code.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment forgejo --replicas=0 -n forgejo
  ```

- Start container
  ```
  kubectl scale deployment forgejo --replicas=1 -n forgejo
  ```

- Inspect container
  ```
  kubectl exec -it -n forgejo deployment/forgejo -- ps -ef
  kubectl exec -it -n forgejo deployment/forgejo -- forgejo --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n forgejo
  ```

- Inspect logs
  ```
  kubectl logs -n forgejo deployment/forgejo --tail=100 -f
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

- [CoreDNS](https://forgejo.org)
