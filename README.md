# Crossplane Demo
### [https://crossplane.io]
## Demo for Course YCIT020 - McGill University Student Project
### Below the Demos Basic Steps:

1. Installing and configuring Crossplane inside a GCP cluster
2. Provision Infrastructure per Crossplane pre-made packages, Crossplane yaml manifest
3. Create another cluster using Crossplane
4. Test Scenario
   - Update node configraton and see Crossplane in action
   - Execute the same with Argos (under Constrcution)
5. Clean-up 

### Assumption: Argos and Helm must be installed and configured already in your cluster
- Helm: [https://helm.sh/]
- Argos: [https://argoproj.github.io/]  
### Pre-Requirements
- Open GCP CLI, in the command promt
- Setting up a GCP cluster: 
- Create cluster: <br />
$ gcloud container clusters create crossplane-demo \
--region us-central1 \\ \
--enable-ip-alias \\ \
--enable-network-policy \\ \
--num-nodes 1 \\ \
--machine-type "e2-standard-2" \\ \
--release-channel stable
- Get credantials for container: 
  $ gcloud container clusters get-credentials crossplane-demo --region us-central1 

### 1 - Installing and Configuring Crossplane
- setup Project ID: $ PROJECT_ID=chrisboullosa-notepad-dev-864 ... please use your Student ID  <br />
- Create namespace: $ kubectl create namespace crossplane-system
- Helm Charts preparation
  - $ helm repo add crossplane-stable https://charts.crossplane.io/stable
  - $ helm repo update
  - $ helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
  - Validating chart and name space
    - $ helm list -n crossplane-system
    - $ kubectl get all -n crossplane-system
- Installing Crossplane CLI:
  - $ curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
  - Bringing to usr/bin directory: $ sudo mv kubectl-crossplane /usr/bin
  - Testing binary: $ kubectl crossplane --help
- Install Configuration Package:
  - $ kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-gcp:v1.4.1 
  - validate package: $ watch kubectl get pkg
- Get GCP Account Keyfile
  - Seeting variables
    - $ PROJECT_ID=chrisboull-notepad-dev-864 
    - $ NEW_SA_NAME=test-service-account-name 
  - #### create service account
    - $ SA="${NEW_SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
    - $ gcloud iam service-accounts create $NEW_SA_NAME --project $PROJECT_ID
  - #### enable cloud API
    - $ SERVICE="sqladmin.googleapis.com"
    - $ gcloud services enable $SERVICE --project $PROJECT_ID
  - #### grant access to cloud API
    - $ ROLE="roles/cloudsql.admin"
    - $ gcloud projects add-iam-policy-binding --role="$ROLE" $PROJECT_ID --member "serviceAccount:$SA"
  - #### create service account keyfile
    - $ gcloud iam service-accounts keys create creds.json --project $PROJECT_ID --iam-account $SA
- Create a Provider Secret
    - $ kubectl create secret generic gcp-creds -n crossplane-system --from-file=creds=./creds.json
-  Configure the Provider: ... copy and pasted in the command prompt:
<p>
	echo "apiVersion: gcp.crossplane.io/v1beta1  <br />
 	kind: ProviderConfig <br />
   metadata: <br />
   &ensp;&ensp;name: default <br />
   spec: <br />
   &ensp;&ensp;projectID: ${PROJECT_ID} <br />
   &ensp;&ensp;credentials: <br />
   &ensp;&ensp;&ensp;&ensp;source: Secret <br />
   &ensp;&ensp;&ensp;&ensp;secretRef: <br />
   &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;namespace: crossplane-system <br />
   &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;name: gcp-creds<br />
   &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key: creds" | kubectl apply -f - <br /> 
    </p> 

### 2 - Provision Infrastructure per Crossplane pre-made packages
- Claim Your Infrastructure:
  - $ kubectl apply -f https://raw.githubusercontent.com/crossplane/crossplane/release-1.4/docs/snippets/compose/claim-gcp.yaml
- Validate: 
  - $ kubectl get postgresqlinstance my-db ...(repeat multiple times)
- To watch your provisioned resources become ready:
  - $ kubectl get crossplane -l crossplane.io/claim-name=my-db
- Describe command: $ kubectl describe secrets db-conn 
  - $ kubectl describe secrets db-conn ... sample
- Consume Your Infrastructure:
  - $ kubectl apply -f https://raw.githubusercontent.com/crossplane/crossplane/release-1.4/docs/snippets/compose/pod.yaml
  - Sample outpout:
  - ![image](https://user-images.githubusercontent.com/72282458/133360681-97b03fd1-3171-4192-86a7-2ea624150a33.png)
- Validate PODS: $ kubectl get pods 

### 3 - Creating a Cluster with Crossplane
- Git clone:
  - $ git clone https://github.com/IntelligentLlama/Crossplanedemo.git
  - $ cd Crossplanedemo
  - $ cp cluster.yaml cluster.yaml 
    - (or copy *.* to target directory .i.e /118918242-notepad)
 - Craete new namespace and SA Account
   -  $ kubectl create namespace team-a
   -  $ export NEW_SA_NAME=devops-toolkit 
 - Repeat STEP 1: Helm Charts preparation and then execute:
   - $ kubectl apply --filename definition.yaml
 - Repeat STEP 1: Get GCP Account Keyfile
 - Repeat STEP 1: Create a Provider Secret
 - Repeat STEP 1: Configure the Provider
   - Run: $ kubectl apply --filename gcp.yaml 
- Creating Infrastructure: the cluster
  - $ kubectl apply --filename cluster.yaml
- Defining composites:
  - $ kubectl describe composition cluster-google
  - $ kubectl explain compositekubernetescluster --recursive
- Resource statuses
  - $ kubectl get compositekubernetesclusters
  - $ kubectl describe compositekubernetescluster team-a
  - $ kubectl get gkeclusters,nodepools
  - $ kubectl get compositekubernetesclusters
  - ... Wait until all it's up and running 
- Accessing the new infrastructure ... also check via GCP-GKE Console (upi know where to go) 
<p>
&ensp;&ensp; $ kubectl --namespace team-a \  <br />
&ensp;&ensp;&ensp;&ensp;get secret cluster \  <br />
&ensp;&ensp;&ensp;&ensp;--output jsonpath="{.data.kubeconfig}" \ <br />
&ensp;&ensp;&ensp;&ensp;| base64 -d \ <br />
&ensp;&ensp;&ensp;&ensp;| tee kubeconfig.yaml <br /> </p>

- Validating nodes and namespaces  
  - $ export KUBECONFIG=$PWD/kubeconfig.yaml
  - $ kubectl get nodes
  - $ kubectl get namespaces
  - $ unset KUBECONFIG
#### Updating infrastructure ...
- Open cluster.yaml in an editor and uncomment `spec.parameters.minNodeCount`
  - Im this sample we change the Min number of Node Counts
- $ kubectl apply --filename cluster.yaml
- $ export KUBECONFIG=$PWD/kubeconfig.yaml
- $ kubectl get nodes
#### Bingo ! changes were pushed via Crossplane

### 5 - Clean-Up
- Cleaning Crossplane stuff:
  - $ unset KUBECONFIG
  - $ kubectl delete --filename cluster.yaml 
  - $ kubectl get compositekubernetesclusters
  - $ kubectl get clusters
  - $ kubectl get clusters,nodegroup,iamroles,iamrolepolicyattachments,vpcs,securitygroups,subnets,internetgateways,routetables
- Cleaning the rest: 
  - $ kubectl delete pod see-db
  - $ kubectl delete postgresqlinstance my-db
  - Uninstall Packages
    - kubectl delete configuration.pkg gcp
    - kubectl delete providerconfig.aws --all
    - kubectl delete provider.pkg gcp	
  - $ helm delete crossplane --namespace crossplane-system
  - $ kubectl delete namespace crossplane-system
  - Helm does not delete CRD objects. You can delete the ones Crossplane created with the following commands:
    - $ kubectl get crd -o name | grep crossplane.io | xargs kubectl delete
  - $ gcloud container clusters delete crossplane-demo --region us-central1
 
 ### The End
