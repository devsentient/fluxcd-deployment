# fluxcd-deploy-test

## Create Namespaces

kubectl apply -f ./namespaces/namespaces.yaml

## AWS ACCESS KEYS

```
export AWS_KEY_ID=$(cat ./keys/aws_access_key | base64 )
export AWS_SECRET_KEY=$(cat ./keys/aws_secret_access_key | base64 )

cp ./secrets/aws_access_key_secret.yaml ./

sed -i '' -E "s#AWS_ACCESS_KEY_ID#${AWS_KEY_ID}#g" aws_access_key_secret.yaml
sed -i '' -E "s#AWS_SECRET_ACCESS_KEY_ID#${AWS_SECRET_KEY}#g" aws_access_key_secret.yaml

kubectl apply -f ./aws_access_key_secret.yaml

rm aws_access_key_secret.yaml
```

## SSH KEYS

### SSH PORTAL

```
ssh-keygen -o -a 100 -t ed25519 -N '' -f ./keys/id_ed25519

export SSH_PORTAL_KEY=$(cat ./keys/id_ed25519 | base64 )
export SSH_PORTAL_PUB_KEY=$(cat ./keys/id_ed25519.pub | base64 )

cp ./secrets/ssh_portal_secret.yaml ./

sed -i '' -E "s#SSH_PORTAL_KEY#${SSH_PORTAL_KEY}#g" ssh_portal_secret.yaml
sed -i '' -E "s#SSH_PORTAL_PUB_KEY#${SSH_PORTAL_PUB_KEY}#g" ssh_portal_secret.yaml

kubectl apply -f ./ssh_portal_secret.yaml

rm ssh_portal_secret.yaml
```

### Flux Git Repo

```
ssh-keygen -t rsa -N '' -f ./keys/id_rsa_github

add .pub key to git repo with shakudo helmchart

export GITHUB_KEY=$(cat ./keys/id_rsa_github | base64 )
export KNOWN_HOSTS=$(cat ./keys/known_hosts | base64 )

cp ./secrets/fluxcd_source_git_secret.yaml ./

sed -i '' -E "s#GITHUB_KEY#${GITHUB_KEY}#g" fluxcd_source_git_secret.yaml
sed -i '' -E "s#KNOWN_HOSTS#${KNOWN_HOSTS}#g" fluxcd_source_git_secret.yaml

kubectl apply -f ./fluxcd_source_git_secret.yaml

rm fluxcd_source_git_secret.yaml
```

### Workloads and Development Git Repo

```
ssh-keygen -t rsa -N '' -f ./keys/id_rsa_workloads_github

add .pub key to your workloads and development git repo

export GITHUB_PUB_KEY=$(cat ./keys/id_rsa_workloads_github.pub | base64 )
export GITHUB_KEY=$(cat ./keys/id_rsa_workloads_github | base64 )

cp ./secrets/workloads_git_server_secret.yaml ./

sed -i '' -E "s#GITHUB_KEY#${GITHUB_KEY}#g" workloads_git_server_secret.yaml
sed -i '' -E "s#GITHUB_PUB_KEY#${GITHUB_PUB_KEY}#g" workloads_git_server_secret.yaml
sed -i '' -E "s#KNOWN_HOSTS#${KNOWN_HOSTS}#g" workloads_git_server_secret.yaml

kubectl apply -f ./workloads_git_server_secret.yaml

rm workloads_git_server_secret.yaml
```

## Certs

```
export DOMAIN_NAME="pat-test.canopyhub.io"

cp ./certs/certificate-hyperplane-istio-istio-wc.yaml ./
cp ./certs/certificate-istio-system-istio-wc.yaml ./

sed -i '' "s#<DOMAIN_NAME>#${DOMAIN_NAME}#g" certificate-hyperplane-istio-istio-wc.yaml
sed -i '' "s#<DOMAIN_NAME>#${DOMAIN_NAME}#g" certificate-istio-system-istio-wc.yaml

kubectl apply -f ./certificate-hyperplane-istio-istio-wc.yaml
kubectl apply -f ./certificate-istio-system-istio-wc.yaml

rm certificate-hyperplane-istio-istio-wc.yaml
rm certificate-istio-system-istio-wc.yaml
```

## Scale up oauth2 proxy
