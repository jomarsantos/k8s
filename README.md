# Description
A simple setup for Kubernetes (k8s).

# Local Setup

## Kubernetes Configuration

This part depends on the service you're using to maintain your k8s cluster and nodes.

### Digital Ocean
0. Install `doctl` - Digital Ocean CLI ([link](https://github.com/digitalocean/doctl))
0. Run `doctl kubernetes cluster kubeconfig save <cluster_name>` where `cluster_name` is the name of the cluster you created above. This will download the k8s config file from DO and sets up `~/.kube/config` accordingly.

## Kubernetes Setup
0. Install `kubectl` - Kubernetes CLI ([link](https://kubernetes.io/docs/tasks/tools/install-kubectl/))
0. Run `kubectl config get-contexts` to see all the contexts that exist in your `kubectl`
  0. Run `kubectl config current-context` whenever you want to figure out what your context you're currently using
0. To switch over to the context related to the cluster you created, run `kubectl config use-context <context_name>` where `context_name` comes from the command ran in the previous step

# Docker Registry Setup

After setting up your registry with whatever service you choose, run the follow command to configure access to the registry from the k8s cluster.

`kubectl create secret docker-registry <secret_name> --docker-server=<registry_server> --docker-username=<username> --docker-password=<password> --docker-email=<email>`

## Notes

- At the time of writing, GitLab seems to be the most convenient (private and unlimited) and price friendly registry
- A k8s deployment will need to have `imagePullSecrets` defined with the value of `secret_name` to utilize the secret and be able to pull images

# Ingress

The main usage of ingress (at least for this setup) is to route domains to the correct service within the k8s cluster so that a request is forwarded to a correct pod.

## Installation

`helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`
`helm install <release_name> stable/nginx-ingress --set controller.publishService.enabled=true`

- This will setup the ingress controller, including a service of type LoadBalancer

## Single Node Cluster w/o Load Balancer

If you're just testing out k8s and are on a budget, there's a couple things you can do. In this scenario, you run a single node within the cluster without a load balance in front of it (since that comes with a cost as well).

`helm install myingress stable/nginx-ingress -f myingress.values.yml`

- This will setup the ingress controller, including a service of type ClusterIP
- Since our ingress controller isn't exposed to the public, we'll need to take an extra step of exposing ports so that it is accessible
- This works by exposing the ports directly on the k8s node's network interfaces

The next step is to add a DNS rule that points your domain to the IP address of the single k8s node

# TLS / Let's Encrypt / Cert-Manager
- Install the CustomResourceDefinitions and cert-manager itself:
  - `kubectl create namespace cert-manager`
  - `helm repo add jetstack https://charts.jetstack.io`
  - `helm repo update`
  - `helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.1 --set installCRDs=true`
  - `kubectl create -f tls/staging_issuer.yaml`
  - `kubectl create -f tls/prod_issuer.yaml`

# Deploying a Service

- Clone the files within `services` and tailor them with the details of your service
- `Makefile`
  - `build`: builds your service's docker image and pushes it to your registry
  - `deploy`: deploys your service to k8s
  - `bd`: builds, pushes and deploys
- Certificates can take awhile, you can see progress with:
  - `kubectl describe ingress`
  - `kubectl describe certificate`
  - `kubectl describe challenges`

## References
- [DO Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes)

# Helpful Tips
- Interactive Bash in a Pod:
  - `kubectl exec --stdin --tty <pod_name> -- /bin/bash`
