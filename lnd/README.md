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

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#local-path-provisioner)

#### traefik

> [!NOTE]
> Instructions are located in the main [README.md](../README.md#traefik)

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
