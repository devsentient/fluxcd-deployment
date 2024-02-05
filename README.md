# fluxcd-deploy-test

## Prerequisites

- install [kubectl](https://kubernetes.io/docs/tasks/tools/)
- install [jq](https://stedolan.github.io/jq/download/)
- Download the [keycloak server distribution](https://www.keycloak.org/downloads) and [update your path](https://www.keycloak.org/docs/latest/server_admin/#admin-cli)
- install [k9s](https://k9scli.io/topics/install/)

## Setup Flux

```
brew install fluxcd/tap/flux

export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

flux check --pre
```

## Create Namespaces

kubectl apply -f ./namespaces/namespaces.yaml

## Configure Secrets

### AWS access keys (for cluster autoscaler)

```
export AWS_KEY_ID=$(cat ./keys/aws_access_key | base64 )
export AWS_SECRET_KEY=$(cat ./keys/aws_secret_access_key | base64 )

cp ./secrets/aws_access_key_secret.yaml ./

sed -i '' -E "s#AWS_ACCESS_KEY_ID#${AWS_KEY_ID}#g" aws_access_key_secret.yaml
sed -i '' -E "s#AWS_SECRET_ACCESS_KEY_ID#${AWS_SECRET_KEY}#g" aws_access_key_secret.yaml

kubectl apply -f ./aws_access_key_secret.yaml

rm aws_access_key_secret.yaml
```

### SSH PORTAL (ssh access into sessions)

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

### Flux Git Repo source (for shakudo helm chart)

```
ssh-keygen -t rsa -N '' -f ./keys/id_rsa_github

export GITHUB_KEY=$(cat ./keys/id_rsa_github | base64 )
export KNOWN_HOSTS=$(cat ./keys/known_hosts | base64 )

cp ./secrets/fluxcd_source_git_secret.yaml ./

sed -i '' -E "s#GITHUB_KEY#${GITHUB_KEY}#g" fluxcd_source_git_secret.yaml
sed -i '' -E "s#KNOWN_HOSTS#${KNOWN_HOSTS}#g" fluxcd_source_git_secret.yaml

kubectl apply -f ./fluxcd_source_git_secret.yaml

rm fluxcd_source_git_secret.yaml
```

> **Note:** The public key generated above will need to be added as a deploy key to the shakudo helm chart repository

### Workloads and Development Git Repo

```
ssh-keygen -t rsa -N '' -f ./keys/id_rsa_workloads_github

export GITHUB_PUB_KEY=$(cat ./keys/id_rsa_workloads_github.pub | base64 )
export GITHUB_KEY=$(cat ./keys/id_rsa_workloads_github | base64 )

cp ./secrets/workloads_git_server_secret.yaml ./

sed -i '' -E "s#GITHUB_KEY#${GITHUB_KEY}#g" workloads_git_server_secret.yaml
sed -i '' -E "s#GITHUB_PUB_KEY#${GITHUB_PUB_KEY}#g" workloads_git_server_secret.yaml
sed -i '' -E "s#KNOWN_HOSTS#${KNOWN_HOSTS}#g" workloads_git_server_secret.yaml

kubectl apply -f ./workloads_git_server_secret.yaml

rm workloads_git_server_secret.yaml
```

> **Note:** The public key generated above will need to be added as a deploy key to your development and workloads source repository

## Install Flux onto your cluster and trigger the helm install

```
export EKS_CONTEXT="replace with arn for eks cluster"
export GIT_REPO_OWNER="replace me" example: devsentient
flux bootstrap github --context=${EKS_CONTEXT} --owner=${GIT_REPO_OWNER} --repository=${GITHUB_REPO} --branch=main --path=clusters/shakudo-eks --personal=false
```

> **Note:** View the status of your install by inspecting the helmrelease resource in k9s

## Add DNS Records (gcloud example)

```
gcloud dns --project=hyperplane-test record-sets create my_cluster.canopyhub.io. --zone=canopyhub --type=CNAME --ttl=300 --rrdatas=my_cluster.us-east-1.elb.amazonaws.com
gcloud dns --project=hyperplane-test record-sets create "*.my_cluster.canopyhub.io." --zone=canopyhub --type=CNAME --ttl=300 --rrdatas=my_cluster.us-east-1.elb.amazonaws.com
```

## Apply Certs (using cert manager)

```
export DOMAIN_NAME="my_cluster.canopyhub.io"

cp ./certs/certificate-hyperplane-istio-istio-wc.yaml ./
cp ./certs/certificate-istio-system-istio-wc.yaml ./

sed -i '' "s#<DOMAIN_NAME>#${DOMAIN_NAME}#g" certificate-hyperplane-istio-istio-wc.yaml
sed -i '' "s#<DOMAIN_NAME>#${DOMAIN_NAME}#g" certificate-istio-system-istio-wc.yaml

kubectl apply -f ./certificate-hyperplane-istio-istio-wc.yaml
kubectl apply -f ./certificate-istio-system-istio-wc.yaml

rm certificate-hyperplane-istio-istio-wc.yaml
rm certificate-istio-system-istio-wc.yaml
```

## Scale up oauth2 proxy (scaled down for fluxcd deployment health checks)

```
kubectl scale deployment/oauth2-proxy --replicas=1 -n hyperplane-core
```

## Configure Keycloak Basic Auth

- [Keycloak setup guide](https://www.notion.so/shakudo/Keycloak-Basic-Auth-Setup-d624e71f6207449c8e3091be94eaa22a)

## Okta Setup Guide

- [Keycloak with Okta SAML Identity Provider](https://www.notion.so/shakudo/Keycloak-with-Okta-SAML-Identity-Provider-ab3e0871cd6648e7b3085340f8831d90)
- [Login to Shakudo from within Okta](https://www.notion.so/shakudo/Login-to-Shakudo-from-within-Okta-d641fb8a03aa430ba74951ac5863e3c5)

