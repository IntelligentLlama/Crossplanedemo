# Crossplane Demo
### [https://crossplane.io]
## Demo for Course YCIT020 - McGill University Student Project
### Below the Demos Basic Steps:

1. Installing and configuring Crossplane inside a GCP cluster
2. Provision Infrastructure per Crossplane pre-made packages, Crossplane yaml manifest
3. Create another cluster using Crossplane
4. Test Drifting scenario
   - Claim your infrastructure: Modifying cluster node zones manually via GCP console
   - Monitor the change and see how Crossplane brings the zones back to original manifest
   - Push the wanted changes with github and see Crossplane in action

### Assumption: Argos and Helm must be installed and configured already in your cluster
- Helm: [https://helm.sh/]
- Argos: [https://argoproj.github.io/]  
### 1 - Installing and Configuring Crossplane
- Open GCP CLI, in the command promt
- Setting up a GCP cluster: 
PROJECT_ID=chrisboullosa-notepad-dev-864 ... please use your Student ID

$ gcloud container clusters create crossplane-demo \
--region us-central1 \\ \
--enable-ip-alias \\ \
--enable-network-policy \\ \
--num-nodes 1 \\ \
--machine-type "e2-standard-2" \\ \
--release-channel stable
- Get credantials for container: 
  $ gcloud container clusters get-credentials crossplane-demo --region us-central1 
- Create namespace: $ kubectl create namespace crossplane-system
- Helm Charts preparation \
  - $ helm repo add crossplane-stable https://charts.crossplane.io/stable
  - $ helm repo update
  - $ helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
- Validating chart and name space
  - $ helm list -n crossplane-system
  - $ kubectl get all -n crossplane-system
- Installing crossplane Master
  - $ curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
- Bringing to usr/bin directory: $ sudo mv kubectl-crossplane /usr/bin
- Testing binary: $ kubectl crossplane --help
- Install configuration registry
  - $ kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-gcp:v1.4.1
- validate package: $ watch kubectl get pkg
- Seeting variables
  - $ PROJECT_ID=chrisboull-notepad-dev-864
  - $ NEW_SA_NAME=test-service-account-name
-  
