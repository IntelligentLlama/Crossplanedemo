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

