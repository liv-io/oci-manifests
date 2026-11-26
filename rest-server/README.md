# rest-server

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
    - [rest-server](#rest-server)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `rest-server`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[rest-server](https://github.com/liv-io/oci-containers/tree/main/rest-server)|Container|`0.15.0-1`|
|[traefik-helm-chart](https://github.com/traefik/traefik-helm-chart)|Helm|`43.0.0`|

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
    name: rest-server
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

#### rest-server

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/rest-server
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/rest-server/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//rest-server?ref=main

  images:
    - name: rest-server
      newName: code.liv.io/liv/rest-server
      newTag: 0.15.0-1

  patches:
    - patch: |-
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: data
        spec:
          resources:
            requests:
              storage: 1000Gi

    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: rest-server
        spec:
          template:
            spec:
              imagePullSecrets:
                - name: code.liv.io

    - patch: |-
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: rest-server
        spec:
          tls:
            - hosts:
                - backup.liv.io
          rules:
            - host: backup.liv.io
              http:
                paths:
                  - path: /
                    pathType: Prefix
                    backend:
                      service:
                        name: rest-server
                        port:
                          number: 8080
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/rest-server/
  kubectl apply --dry-run=server -k ./apps/rest-server/
  kubectl apply -k ./apps/rest-server/
  ```

- Validate deployment
  ```
  kubectl rollout status -n rest-server deployment/rest-server
  kubectl logs -n rest-server deployment/rest-server --tail=100
  ```

- Get `rest-server` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://rest-server.liv.io/healthz
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment rest-server --replicas=0 -n rest-server
  ```

- Start container
  ```
  kubectl scale deployment rest-server --replicas=1 -n rest-server
  ```

- Inspect container
  ```
  kubectl exec -it -n rest-server deployment/rest-server -- ps -ef
  kubectl exec -it -n rest-server deployment/rest-server -- rest-server --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n rest-server
  ```

- Inspect logs
  ```
  kubectl logs -n rest-server deployment/rest-server --tail=100 -f
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

- [rest-server](https://github.com/restic/rest-server)
