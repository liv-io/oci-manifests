# larix

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
    - [larix](#larix)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `larix`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[larix](https://github.com/liv-io/oci-containers/tree/main/larix)|Container|`0.1.0`|
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
    name: larix
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

#### larix

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/larix
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/larix/kustomization.yaml
  EOF
  ```

- Validate deployment
  ```
  kubectl rollout status -n larix deployment/larix
  kubectl logs -n larix deployment/larix --tail=100
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment larix --replicas=0 -n larix
  ```

- Start container
  ```
  kubectl scale deployment larix --replicas=1 -n larix
  ```

- Inspect container
  ```
  kubectl exec -it -n larix deployment/larix -- ps -ef
  kubectl exec -it -n larix deployment/larix -- larix --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n larix
  ```

- Inspect logs
  ```
  kubectl logs -n larix deployment/larix --tail=100 -f
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

- [larix](https://github.com/liv-io/oci-containers/tree/main/larix)
