# coredns

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
    - [coredns](#coredns)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `coredns`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[coredns](https://github.com/liv-io/oci-containers/tree/main/coredns)|Container|`1.14.7-1`|
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
    name: coredns
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

#### coredns

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/coredns
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/coredns/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@github.com:liv-io/oci-manifests.git//coredns?ref=main

  images:
    - name: coredns
      newName: code.liv.io/liv/coredns
      newTag: 0.14.6-4

  configMapGenerator:
    - name: zones
      behavior: replace
      files:
        - zones/db.asdf.com.
        - zones/db.1.1.1.in-addr.arpa.

  patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: coredns
        spec:
          template:
            spec:
              imagePullSecrets:
                - name: code.liv.io

    - patch: |-
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: coredns
        spec:
          tls:
            - hosts:
                - edge-dns.liv.io
          rules:
            - host: edge-dns.liv.io
              http:
                paths:
                  - path: /
                    pathType: Prefix
                    backend:
                      service:
                        name: coredns
                        port:
                          number: 8080
  EOF
  ```

- Create `zones` directory
  ```
  install --directory --mode=0755 ./apps/coredns/zones
  ```

- Populate zone files

> [!TIP]
> For reference, consult example [zone](https://github.com/liv-io/oci-manifests/tree/main/coredns/zones) files

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/coredns/
  kubectl apply --dry-run=server -k ./apps/coredns/
  kubectl apply -k ./apps/coredns/
  ```

- Validate deployment
  ```
  kubectl rollout status -n coredns deployment/coredns
  kubectl logs -n coredns deployment/coredns --tail=100
  ```

- Get `coredns` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://edge-dns.liv.io/health
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment coredns --replicas=0 -n coredns
  ```

- Start container
  ```
  kubectl scale deployment coredns --replicas=1 -n coredns
  ```

- Inspect container
  ```
  kubectl exec -it -n coredns deployment/coredns -- ps -ef
  kubectl exec -it -n coredns deployment/coredns -- coredns --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n coredns
  ```

- Inspect logs
  ```
  kubectl logs -n coredns deployment/coredns --tail=100 -f
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

- [CoreDNS](https://coredns.io)
