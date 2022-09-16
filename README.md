# vault-injector-k8s
```
# helm install vault hashicorp/vault --set "injector.externalVaultAddr="
# VAULT_HELM_SECRET_NAME=$(kubectl get secrets --output=json -n ns | jq -r '.items[].metadata | select(.name|startswith("vault-injector-token")).name')
# TOKEN_REVIEW_JWT=$(kubectl get secret $VAULT_HELM_SECRET_NAME --output='go-template={{ .data.token }}' -n ns | base64 --decode)
# KUBE_HOST=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.server}')
# KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}'  | base64 --decode)
# vault login -address=https://vault -method=userpass username= password=''
# export VAULT_ADDR="https://vault"
# vault auth enable kubernetes
# vault policy write devwebapp - <<EOF          
path "secret/env/my-secret/*" {
capabilities = ["read","list"]
}
EOF
# vault write auth/kubernetes/config \         
token_reviewer_jwt="$TOKEN_REVIEW_JWT" \  
kubernetes_host="$KUBE_HOST" \                
kubernetes_ca_cert="$KUBE_CA_CERT"
# vault write auth/kubernetes/role/devweb-app \
bound_service_account_names=internal-app \
bound_service_account_namespaces= \
policies=devwebapp \              
ttl=24h

```
# Multiple K8s cluster with vault agent
```
#  helm install vault hashicorp/vault --set "injector.externalVaultAddr=https://vault.domain.io/" --set "injector.authPath=auth/prod-cluster"
#  vault auth enable --path prod-copen kubernetes
#  vault write auth/dev-cluster/role/devweb-app \
bound_service_account_names="*" \
bound_service_account_namespaces="*" \
policies=devwebapp \
ttl=24h
```


# Đối với kubenetes version 1.21+ 
```vault write auth/kubernetes/config \         
token_reviewer_jwt="$TOKEN_REVIEW_JWT" \  
kubernetes_host="$KUBE_HOST" \           
kubernetes_ca_cert="$KUBE_CA_CERT" \
issuer="https://container.googleapis.com/v1/projects/cmc-lab/locations/asia-southeast1-a/clusters/dungla-test-vault-01"
```

*issuser là trường iss trong đoạn sau:*
```
kubectl proxy &                   
curl --silent http://127.0.0.1:8001/api/v1/namespaces/default/serviceaccounts/default/token \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"apiVersion": "authentication.k8s.io/v1", "kind": "TokenRequest"}' \
  | jq -r '.status.token' \
  | cut -d . -f2 \
  | base64 -d
```

# helm cmc app
```
helm repo add helm-cmc https://nexus.io/repository/helm-cmc/ --username dev --password '2'
helm package deploy
curl -is -u 'dung':'pass' https://nexus.io/repository/helm-cmc/ --upload-file cmc.tgz
```
- test:
```helm template test helm-cmc/cmc-app -f k8s/values.yml```
# Reference
- https://devopscube.com/vault-agent-injector-tutorial/
- https://learn.hashicorp.com/tutorials/vault/kubernetes-external-vault?in=vault/kubernetes
- https://www.vaultproject.io/docs/auth/kubernetes#discovering-the-service-account-issuer
- lỗi: https://github.com/hashicorp/vault-helm/issues/562
