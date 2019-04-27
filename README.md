# gangwaydexdemo
Proof of Concept instructions for OIDC authentication on minikube using gangway/dex.  This tutorial uses NodePorts to access dex/gangway for the sake of simplicity in a local development environment - Ingress and Loadbalancer are recommended in production.

[Jump to the Tutorial](https://github.com/danielorf/gangwaydexdemo#instructions)

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



## Procedure
### Architecture Notes
- Kubernetes v1.14.0 (minikube)
- Dex deployed to Kubernetes with port:nodeport of 5556:32000
- Gangway deployed to Kubernetes with port:nodeport of 8080:32001

### Other Important Notes
- The instructions and configs assume that dex and gangway are reachable through these hostnames:
    - dex hostname:  dex.gangwaydexdemo.com
    - gangway hostname:  gangway.gangwaydexdemo.com
- dex and gangway services are available via nodeports.  In a production environment, ingress and loadbalancer would be recommended instead.
- dex OIDC config http endlpoint:  https://dex.gangwaydexdemo.com:32000/dex/.well-known/openid-configuration

### Instructions
1. Deploy minikube: `minikube start`
2. Generate certs to match hostnames of dex and gangway and install on minikube
    - If you choose to generate your own certs:
        - In ./certs/gencrt.sh, modify alt_names to match desired dex and gangway hostames.  It is currently configured for the hostnames mentioned above.
        - Rename TLS cert to dex.crt, key to dex.key and CA cert to dex-ca.crt
    - Otherwise, example certs are included in ./certs
    - SSH into minikube (`minikube ssh`) and place all 3 in /etc/dex/pki
3. /etc/hosts modifications
    - On minikube and dev machine, add entries to /etc/hosts to point chosen dex/gangway hostnames (above) to the minikube IP address
    - On the dev machine only, add entry to point kubernetes.default.svc.cluster.local to the kube-apiserver address which can be found with `echo "https://$(minikube ip):8443"`.
4. Apply dex config `kubectl apply -f configs/dex.yaml`
5. Apply gangway configs and generate secret:
    - `kubectl apply -f configs/gangway-ns.yaml`
    - `kubectl -n gangway create secret generic gangway-key --from-literal=sesssionkey=$(openssl rand -base64 32)`
    - `kubectl apply -f configs/gangway.yaml`
6. Modify kube-apiserver parameters to direct it to dex
    - ssh into master node
    - `sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml`
    - Add the following to spec.containers.command:
        ```
        - --oidc-issuer-url=https://dex.gangwaydexdemo.com:32000/dex
        - --oidc-client-id=gangway
        - --oidc-ca-file=/etc/dex/pki/dex-ca.crt
        - --oidc-username-claim=email
        - --oidc-groups-claim=groups
        ```
    - Add the following to spec.containers.volumeMounts:
        ```
        - mountPath: /etc/dex/pki
        name: dex-certs
        readOnly: true
        ```
    - Add the following to spec.volumes:
        ```
        - hostPath:
            path: /etc/dex/pki
            type: DirectoryOrCreate
            name: dex-certs
        ```
    - The kube-apiserver container should restart after modification.  Ensure that is does by running `docker ps` from the master node to check if the container is new.
7. Modify coredns configmap to map external hostnames internally
    - `kubectl edit cm coredns -n kube-system`
    - Add the following to data.Corefile..:53 after 'health'
        - `rewrite name gangway.gangwaydexdemo.com gangwaysvc.gangway.svc.cluster.local`
        - `rewrite name dex.gangwaydexdemo.com dex.default.svc.cluster.local`
    - Restart all coredns pods
        - `kubectl delete pod coredns-XXXXXXX -n kube-system`
8. Add a rolebinding for the new user:
    - `kubectl create rolebinding test-admin-binding --clusterrole=admin --user=admin@example.com --namespace=kube-system`
    - Note that this command gives admin access to the user, not recommended for production use.
9. Begin the dex/gangway login process.  It's recommended that you back up and delete your current KUBECONFIG before running the `kubectl` commands below as they will overwrite it.
    - Visit http://gangway.gangwaydexdemo.com:32001
        - Note that login occurs in a dex portal which is https
    - Click 'Sign In' button
    - Click 'Log in with Email'
    - Enter credentials:  admin@example.com/password
    - Click 'Grant Access'
    - Follow intructions after the line "Once kubectl is installed, you may execute the following:" to add the token to your KUBECONFIG
    - Test access with `kubectl get pods -n kube-system`
