# memos

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
    - [memos](#memos)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `memos`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[memos](https://github.com/liv-io/oci-containers/tree/main/memos)|Container|`26.11.0-1`|
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
    name: memos
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

#### memos

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/memos
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/memos/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//memos?ref=main

  images:
    - name: memos
      newName: code.liv.io/liv/memos
      newTag: 26.10.0-1

  patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: memos
        spec:
          template:
            spec:
              imagePullSecrets:
                - name: code.liv.io

    - patch: |-
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: memos
        spec:
          tls:
            - hosts:
                - memos.liv.io
          rules:
            - host: memos.liv.io
              http:
                paths:
                  - path: /
                    pathType: Prefix
                    backend:
                      service:
                        name: memos
                        port:
                          number: 8081
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/memos/
  kubectl apply --dry-run=server -k ./apps/memos/
  kubectl apply -k ./apps/memos/
  ```

- Validate deployment
  ```
  kubectl rollout status -n memos deployment/memos
  kubectl logs -n memos deployment/memos --tail=100
  ```

- Get `memos` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://memos.liv.io/healthz
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment memos --replicas=0 -n memos
  ```

- Start container
  ```
  kubectl scale deployment memos --replicas=1 -n memos
  ```

- Inspect container
  ```
  kubectl exec -it -n memos deployment/memos -- ps -ef
  kubectl exec -it -n memos deployment/memos -- memos --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n memos
  ```

- Inspect logs
  ```
  kubectl logs -n memos deployment/memos --tail=100 -f
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

- [Memos](https://github.com/usememos/memos)
