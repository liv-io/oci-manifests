# prometheus

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
    - [prometheus](#prometheus)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `prometheus`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[prometheus](https://quay.io/prometheus/prometheus)|Container|`3.15.0`|
|[local-path-provisioner](https://github.com/rancher/local-path-provisioner)|Kustomize|`0.0.37`|
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
    name: prometheus
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

#### prometheus

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/prometheus
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/prometheus/kustomization.yaml
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/prometheus/
  kubectl apply --dry-run=server -k ./apps/prometheus/
  kubectl apply -k ./apps/prometheus/
  ```

- Validate deployment
  ```
  kubectl rollout status -n prometheus deployment/prometheus
  kubectl logs -n prometheus deployment/prometheus --tail=100
  ```

- Get `prometheus`
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://prometheus.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment prometheus --replicas=0 -n prometheus
  ```

- Start container
  ```
  kubectl scale deployment prometheus --replicas=1 -n prometheus
  ```

- Inspect container
  ```
  kubectl exec -it -n prometheus deployment/prometheus -- ps -ef
  kubectl exec -it -n prometheus deployment/prometheus -- prometheus --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n prometheus
  ```

- Inspect logs
  ```
  kubectl logs -n prometheus deployment/prometheus --tail=100 -f
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

- [prometheus](https://github.com/prometheus/prometheus)
