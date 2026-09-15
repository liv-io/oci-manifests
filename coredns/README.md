# coredns

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
|[local-path-provisioner](https://github.com/rancher/local-path-provisioner)|Kustomize|`0.0.37`|
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

#### local-path-provisioner

> [!IMPORTANT]
> Deploy `local-path-provisioner` if not included in your Kubernetes distribution

- Create manifest directory
  ```
  install --directory --mode=0755 ./infra/local-path-provisioner
  ```

- Populate manifest
  ```
  cat <<EOF > ./infra/local-path-provisioner/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  namespace: local-path-storage

  resources:
    - https://github.com/rancher/local-path-provisioner/deploy?ref=master

  images:
    - name: rancher/local-path-provisioner
      newTag: v0.0.37

  patches:
    - target:
        kind: StorageClass
        name: local-path
      patch: |-
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: local-path
          annotations:
            storageclass.kubernetes.io/is-default-class: "true"
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./infra/local-path-provisioner/
  kubectl apply --dry-run=server -k ./infra/local-path-provisioner/
  kubectl apply -k ./infra/local-path-provisioner/
  ```

- Validate deployment
  ```
  kubectl rollout status -n local-path-storage deployment/local-path-provisioner
  kubectl get all -n local-path-storage -l app=local-path-provisioner
  ```

#### traefik

- Create manifest directory
  ```
  install --directory --mode=0755 ./infra/traefik
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./infra/traefik/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - https://raw.githubusercontent.com/traefik/traefik/refs/tags/v3.7.13/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
    - https://raw.githubusercontent.com/traefik/traefik/v3.7.13/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
    - git@github.com:liv-io/oci-manifests.git//traefik?ref=main

  configMapGenerator:
    - name: internal-ca-cert
      namespace: kube-system
      behavior: replace
      files:
        - ca1.liv.io.crt

  patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: traefik
          namespace: kube-system
        spec:
          template:
            spec:
              containers:
                - name: traefik
                  env:
                    - name: LEGO_CA_CERTIFICATES
                      value: /etc/traefik/ca/ca1.liv.io.crt

    - target:
        kind: Deployment
        name: traefik
        namespace: kube-system
      patch: |-
        - op: add
          path: /spec/template/spec/containers/0/args/-
          value: --certificatesresolvers.myresolver.acme.caserver=https://ca.liv.io/acme/acme/directory
        - op: add
          path: /spec/template/spec/containers/0/args/-
          value: --certificatesresolvers.myresolver.acme.email=admin@liv.io
  EOF
  ```

- Render and apply manifest
  ```
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply --dry-run=client -f -
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply --dry-run=server -f -
  kustomize build ./infra/traefik/ --enable-helm | kubectl apply -f -
  ```

- Validate deployment
  ```
  kubectl rollout status -n kube-system deployment/traefik
  kubectl get all -n kube-system -l app.kubernetes.io/name=traefik
  ```

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

- Render manifest
  ```
  kubectl apply --dry-run=client -k ./apps/coredns/
  kubectl apply --dry-run=server -k ./apps/coredns/
  ```

- Apply manifest
  ```
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
