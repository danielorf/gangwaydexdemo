# gangwaydexdemo
Proof of Concept instructions for OIDC authentication on minikube using gangway/dex

## Goals
- Provide a set of instructions to set up OpenID Connect (OIDC) authentication flow for Kubernetes independent of proprietary tools (e.g. AWS Elastic Loadbalancer) 
- Use [dex](https://github.com/dexidp/dex) and [gangway](https://github.com/heptiolabs/gangway) for auth flow tooling
- Provide example configurations to get up and running quickly
- List helpful resources for setting up and using dex/gangway with Kubernetes

## Resources
### Dex
- Github repo:  https://github.com/dexidp/dex
- "Getting Started" tutorial:  https://github.com/dexidp/dex/blob/master/Documentation/getting-started.md
- Kubernetes authentication through dex:  https://github.com/dexidp/dex/blob/master/Documentation/kubernetes.md
- Dex OIDC config example:  https://github.com/dexidp/dex/blob/master/Documentation/connectors/oidc.md
- Integrating dex in your applications:  https://github.com/dexidp/dex/blob/master/Documentation/using-dex.md
- Dex helm chart:  https://github.com/helm/charts/tree/master/stable/dex
- Example dex config from SUSE CaaSP:  https://github.com/kubic-project/salt/blob/3b2cdb6056112607b139793c07d4c89266aba738/salt/addons/dex/manifests/15-configmap.yaml
### Gangway
- Github repo:  https://github.com/heptiolabs/gangway
- Deployment example with dex:  https://github.com/heptiolabs/gangway/tree/master/docs
- Configuration parameters description:  https://github.com/heptiolabs/gangway/blob/master/docs/configuration.md
- Dex parameter mapping:  https://github.com/heptiolabs/gangway/blob/master/docs/configuration.md
- Alternative gangway/dex tutorial with example configs included:  https://github.com/alexbrand/gangway-dex-tutorial
- Gangway helm chart:  https://github.com/helm/charts/tree/master/stable/gangway
### Minikube
- Github repo:  https://github.com/kubernetes/minikube
- Official docs:  https://kubernetes.io/docs/setup/minikube/
### Other
- Api server access without kubectl proxy:  https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy


### Notes
- Dex OIDC config http endlpoint:  https://{host}:{port}/dex/.well-known/openid-configuration
