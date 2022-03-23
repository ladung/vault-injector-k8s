# vault-injector-k8s
```
helm install vault hashicorp/vault --set "injector.externalVaultAddr="
VAULT_HELM_SECRET_NAME=$(kubectl get secrets --output=json -n ns | jq -r '.items[].metadata | select(.name|startswith("vault-injector-token")).name')
TOKEN_REVIEW_JWT=$(kubectl get secret $VAULT_HELM_SECRET_NAME --output='go-template={{ .data.token }}' -n ns | base64 --decode)
KUBE_HOST=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.server}')
KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}'  | base64 --decode)
vault login -address=https://vault -method=userpass username= password=''
export VAULT_ADDR="https://vault"
vault auth enable kubernetes
vault policy write devwebapp - <<EOF          
path "secret/env/my-secret/*" {
capabilities = ["read","list"]
}
EOF
vault write auth/kubernetes/config \         
token_reviewer_jwt="$TOKEN_REVIEW_JWT" \  
kubernetes_host="$KUBE_HOST" \                
kubernetes_ca_cert="$KUBE_CA_CERT"
vault write auth/kubernetes/role/devweb-app \
bound_service_account_names=internal-app \
bound_service_account_namespaces= \
policies=devwebapp \              
ttl=24h

```
# helm cmc app
```
helm repo add helm-cmc https://nexus.io/repository/helm-cmc/ --username dev --password '2'
helm package deploy
curl -is -u 'dung':'pass' https://nexus.io/repository/helm-cmc/ --upload-file cmc.tgz
```
# Reference
https://devopscube.com/vault-agent-injector-tutorial/
