# lnd

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
    - [lnd](#lnd)
- [Commands](#commands)
- [License](#license)
- [Credits](#credits)
- [Appendix](#appendix)

## About

OCI manifest for `lnd`.

## Dependencies

### Resources

|Name|Type|Version|
|:---|:---|:---|
|[lnd](https://github.com/liv-io/oci-containers/tree/main/lnd)|Container|`0.21.4-beta-1`|
|[local-path-provisioner](https://github.com/rancher/local-path-provisioner)|Kustomize|`0.0.37`|
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
    name: lnd
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -f ./bootstrap/namespaces.yaml
  kubectl apply --dry-run=server -f ./bootstrap/namespaces.yaml
  kubectl apply -f ./bootstrap/namespaces.yaml
  ```

- Define lnd secrets
  ```
  kubectl create secret generic lnd-secrets --namespace=lnd --from-literal=BITCOIND_RPCPASS="U-JL.FNdKEU9Vkc-g9AjLk9rrAH.svev" --from-literal=WALLET_PASSWORD='F3s+rpgwwRukPmf,rtNAPpqdMAVRMV,c' --dry-run=client -o yaml | kubectl apply -f -
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
    - https://raw.githubusercontent.com/traefik/traefik/refs/tags/v3.7.14/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
    - https://raw.githubusercontent.com/traefik/traefik/v3.7.14/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
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

#### lnd

- Create manifest directory
  ```
  install --directory --mode=0755 ./apps/lnd
  ```

- Patch upstream manifest
  ```
  cat <<EOF > ./apps/lnd/kustomization.yaml
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - git@code.liv.io:liv/oci-manifests.git//lnd?ref=main

  images:
    - name: lnd
      newName: code.liv.io/liv/lnd
      newTag: 0.21.4-beta-1

  patches:
    - patch: |-
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: lnd
        spec:
          template:
            spec:
              imagePullSecrets:
                - name: code.liv.io
              containers:
                - name: lnd
                  env:
                    - name: BITCOIN_NETWORK
                      value: "testnet3"
                    - name: ALIAS
                      value: "liv.io"
                    - name: BITCOIND_RPCHOST
                      value: "ext-latest.btc:18332"
                    - name: BITCOIND_ZMQPUBRAWBLOCK
                      value: "tcp://ext-latest.btc:5557"
                    - name: BITCOIND_ZMQPUBRAWTX
                      value: "tcp://ext-latest.btc:5558"
                    - name: COLOR
                      value: "#00cdfa"
                    - name: EXTERNALIP
                      value: "213.193.83.252"

    - patch: |-
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: lnd
        spec:
          tls:
            - hosts:
                - lnd-test.liv.io
          rules:
            - host: lnd-test.liv.io
              http:
                paths:
                  - path: /
                    pathType: Prefix
                    backend:
                      service:
                        name: lnd
                        port:
                          number: 8080
  EOF
  ```

- Render and apply manifest
  ```
  kubectl apply --dry-run=client -k ./apps/lnd/
  kubectl apply --dry-run=server -k ./apps/lnd/
  kubectl apply -k ./apps/lnd/
  ```

- Validate deployment
  ```
  kubectl rollout status -n lnd deployment/lnd
  kubectl logs -n lnd deployment/lnd --tail=100
  ```

- Get `lnd` health
  ```
  curl --proto '=https' --tlsv1.3 --request GET --location https://lnd.liv.io
  ```

## Commands

- Stop container
  ```
  kubectl scale deployment lnd --replicas=0 -n lnd
  ```

- Start container
  ```
  kubectl scale deployment lnd --replicas=1 -n lnd
  ```

- Inspect container
  ```
  kubectl exec -it -n lnd deployment/lnd -- ps -ef
  kubectl exec -it -n lnd deployment/lnd -- lnd --version
  ```

- Show detailed description
  ```
  kubectl describe pod -n lnd
  ```

- Inspect logs
  ```
  kubectl logs -n lnd deployment/lnd --tail=100 -f
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

- [lnd](https://lightning.engineering)
