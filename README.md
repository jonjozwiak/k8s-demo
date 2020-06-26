# k8s-demo

These are steps as shown in my Kubernetes demo.  Clone this repo and use it for your own testing.  

These steps assume you have a Route 53 domain and hosted zone already.  If not, see the AWS Documentation [here](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html)

This also assumes you have a public SSL certificate in ACM that covers your domain name and wildcard domain name (such as example.com and \*.example.com).  These steps are documented [here](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html).  Note the certificate must be in the same region as your planned EKS cluster deployment.  Make note of the ARN for your certificate as it is needed later.  

## Step 1 - Install EKS 

This is covered in AWS documentation [here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html).

### Install eksctl 
 * On Linux 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

 * On Mac
```
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
eksctl version 
brew upgrade eksctl && brew link --overwrite eksctl
```

### Update external-dns in your repo to reflect your domain name

```
vi manifests/kube-system/external-dns.yaml
--domain-filter=Your-Domain.com  
```


### Deploy your cluster

There is already a cluster.yaml template in this directory you can use.  

```
eksctl create cluster -f cluster.yaml
kubectl get nodes
```

### Add Weave Flux for GitOps

Weave Flux runs in a Kubernetes cluster and monitors your git repos for changes.  It will apply manifests as they change.  It can also monitor your Docker registry and deploy any image changes.  

Update te git-url and git-email below to reflect your environment before running. 

```
# Create a deploy key (used by Flux) and upload to your cluster
ssh-keygen -q -N "" -f weave-flux-deploy-key

kubectl create ns flux 
kubectl create secret generic flux-git-deploy --from-file=identity=weave-flux-deploy-key -n flux 

# Install Flux (from eksctl) 
EKSCTL_EXPERIMENTAL=true \
    eksctl enable repo \
        --git-url git@github.com:jonjozwiak/k8s-demo \
        --git-email jon@example.com \
        --git-paths=manifests \
        --cluster eks1 \
        --region us-west-2
```

In GitHub, go to your repo and add a deploy key as follows:

* In GitHub go to 'Settings' for the repo (NOT YOUR OVERALL PROFILE)
 * Click 'Deploy Keys' and 'Add deploy key'
 * Title: Weave Flux Deploy Key
 * Key: <Paste your weave-flux-deploy-key.pub>
 * Click the 'Allow Write Access' and click 'Add Key'

Within 5 minutes this will deploy your base manifests.  

## Demo Steps for a simple Python Flask App 

See the steps in my [hello-python repo](https://github.com/jonjozwiak/hello-python)


## Cleanup 
eksctl delete cluster -f cluster.yaml


