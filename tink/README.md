# tink

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
    - [tink](#tink)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `tink`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[tink](https://github.com/liv-io/tink)|Container|`0.1.10`|
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
    name: tink
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

#### tink

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/tink
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/tink/kustomization.yaml
  EOF
  ```

- Validate deployment
  ```
  kubectl rollout status -n tink deployment/tink
  kubectl logs -n tink deployment/tink --tail=100
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment tink --replicas=0 -n tink
  ```

- Start container
  ```
  kubectl scale deployment tink --replicas=1 -n tink
  ```

- Inspect container
  ```
  kubectl exec -it -n tink deployment/tink -- ps -ef
  kubectl exec -it -n tink deployment/tink -- tink --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n tink
  ```

- Inspect logs
  ```
  kubectl logs -n tink deployment/tink --tail=100 -f
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

- [tink](https://github.com/liv-io/tink)
