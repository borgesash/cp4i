# cp4i
This guide installs Cloud Pak for Integration on an Openshift cluster.

Reference: https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=installing
This install guide is for CP4I v16.1.0. This is the latest SC-2 (long term support) version of Cloud Pak for Integration. For the latest CD release, see [What's new in Cloud Pak for Integration 16.1.2] (https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.2?topic=whats-new-in-cloud-pak-integration-1612)

We will install the following components of CP4I: 
- Queue Manager
- App Connect Enterprise Integration Servers

## Overview
IBM Cloud Pak® for Integration installation consists of a Red Hat® OpenShift® Container Platform cluster with one or more operators installed and one or more deployed instances. You can complete all the installation steps yourself or simplify the process by installing on a managed OpenShift cluster from a cloud provider.

The main installation tasks are as follows:
1.	Add catalog sources, to make the Cloud Pak for Integration operators available to the cluster.
2.	Install the Cloud Pak for Integration operators.
3.	Apply your entitlement key.
4.	Deploy the IBM Cloud Pak Platform UI and other instances.
This guide assumes you already have an OCP cluster with the right version and capacity up and running. The demo is based upon OCP v4.16.x with 5 worker nodes 32 vCPUs X 128 GB memory each. 
Note: IBM Cloud Pak® for Integration 16.1.0 supports Red Hat OpenShift 4.12, 4.14, 4.15, 4.16, 4.17, 4.18, and 4.19. 

## Before you begin
<details closed>

a. Prepare for installation by reviewing the [Planning](https://www.ibm.com/docs/en/SSGT7J_16.1.0/planning/planning.html) section. 
   Begin with these topics:
    - [Operating environment](https://www.ibm.com/docs/en/SSGT7J_16.1.0/planning/operating_environment.html)
    - [Storage considerations](https://www.ibm.com/docs/en/SSGT7J_16.1.0/planning/storage.html)
    - [Considerations for high availability](https://www.ibm.com/docs/en/SSGT7J_16.1.0/planning/high_availability.html)
    - [Structuring your deployment](https://www.ibm.com/docs/en/SSGT7J_16.1.0/planning/deployment_considerations.html) (Separate namespace vs global namespace)
	
b. Make decisions about how you will install the operators:
 
   - Determine which Cloud Pak for Integration operators you need. For more information, see "Operators available to install" in [Installing the operators by using the Red Hat OpenShift console](https://www.ibm.com/docs/en/SSGT7J_16.1.0/install/install_operators.html). For more information about operators in general, see [Operator reference](https://www.ibm.com/docs/en/SSGT7J_16.1.0/reference/operators_reference.html).
   - Decide which installation mode you will use to install the operators. For more information, see [Installing the operators](https://www.ibm.com/docs/en/SSGT7J_16.1.0/install/install_operators_container.html).

c. Install an appropriate OpenShift cluster. 
   For more information, see [Getting started in OpenShift Container Platform](https://www.ibm.com/links?url=https%3A%2F%2Fdocs.redhat.com%2Fen%2Fdocumentation%2Fopenshift_container_platform%2F4.14%2Fhtml%2Fabout%2Fwelcome-index).

d. Tools required: 

   - Install [oc CLI](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/cli_tools/openshift-cli-oc#cli-getting-started)

e. Obtaining your entitlement key

   - Go to the [Container software library](https://myibm.ibm.com/products-services/containerlibrary).
   - For any key that is listed, click Copy.
		Set your entitlement key:
		
      ```
      export ENT_KEY=<my-key>
      ```
 
   - (Optional) Verify the validity of the key by logging in to the IBM Entitled Registry by using a container tool.
      ```
      docker login cp.icr.io --username cp --password entitlement_key
      ```
  
f. Set the correct Storage type

Storage options 
    Keycloak uses either the default storage class in Red Hat OpenShift Container Platform, or the storage class configured in the IBM Cloud Pak® foundational services Kubernetes resource. Before installing instances, do one of the following:
    * Set a default storage class by adding the storageclass.kubernetes.io/is-default-class:'true' annotation in Red Hat OpenShift Container Platform.
    * Specify a storage class name for spec.storageClass in the CommonService resource.

<details closed>
	
   - Identify current storage type
     Run command to identify the existing Storage type:
```
oc get sc
```

  Your will get a response like this showing 'ocs-storagecluster-cephfs (default)' (then proceed with the steps below):

  <img width="1028" height="88" alt="image" src="https://github.com/user-attachments/assets/db44e44f-714f-4162-9dba-fa9ae7bedde0" />

  

For this demo environment, we are using ODF, so default storage will be ocs-storagecluster-ceph-rbd. 

- Remove the existing default storage class

  Create sc-remove-default.yaml with following content

```yaml annotate
cat <<EOF > sc-remove-default.yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
EOF
```

  Execute the follwing command
  ```
  oc get sc | grep default | awk '{system("oc patch storageclass " $1 " --patch-file sc-remove-default.yaml")}'
  ```
  Successful response would look like
  `storageclass.storage.k8s.io/ocs-storagecluster-cephfs patched`

- Add the correct default storage class

  create sc-set-default.yaml with the following content
```yaml annotate
cat <<EOF > sc-set-default.yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
EOF
```

  Execute the following command
  ```
  oc patch storageclass ocs-storagecluster-ceph-rbd --patch-file sc-set-default.yaml
  ```
  Successfull response would look like:
  `storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd patched`

- Validate the default storage class
	Run the following command to verify that the default storage class is correct set to your desired option. In this case it should be ocs-storagecluster-ceph-rbd
```
oc get sc 
```

<img width="1055" height="104" alt="image" src="https://github.com/user-attachments/assets/0c7c632f-a89d-420a-a06e-bc6413153f16" />
</details>
</details>

## Install Steps

### Namespaces
<details closed>
We will be using the All Namespaces mode to install CP4I.
Here is the namespace where each component will be deployed by the end of the installation process.

| namespace      | what is deployed?       |
| -------------- | -------------- |
| openshift-marketplace| All Catalogsources|
| openshift-operators| All Operators|
| ibm-common-services| Common Services (KeyCloak/EDB)|
| cp4i-mq| MQ instances|
| cp4-ace| App Connect Instances|
</details>

### Deploy IBM Foundational Services
<details closed>

Red Hat OpenShift Operators automate the creation, configuration, and management of instances of Kubernetes-native applications. Operators provide automation at every level of the stack—from managing the parts that make up the platform all the way to applications that are provided as a managed service.
Red Hat OpenShift uses the power of Operators to run the entire platform in an autonomous fashion while exposing configuration natively through Kubernetes objects, allowing for quick installation and frequent, robust updates. In addition to the automation advantages of Operators for managing the platform, Red Hat OpenShift makes it easier to find, install, and manage Operators running on your clusters.

The foundational services help you manage and administer IBM software on your cluster. IBM Cloud Pak foundational services component is included in several IBM Cloud Paks.

#### 1. Installing Cert Manager (Required if using APIC/Event Manager/Event Processing)

_Important: The API Connect cluster, Event Manager, and Event Processing instances require you to install an appropriate certificate manager.
OPTIONAL: To install via Openshift Console UI, follow the instructions in [Installing the cert-manager Operator for Red Hat OpenShift](https://docs.openshift.com/container-platform/4.14/security/cert_manager_operator/cert-manager-operator-install.html)_

- Create a namespace

```
oc new-project cert-manager-operator
```

- Create the Operator

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cert-manager-operator
  namespace: cert-manager-operator 
spec:
  targetNamespaces:
  - cert-manager-operator
EOF
```
		
- Create the subscription

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator 
spec:
  channel: "stable-v1" 
  name: openshift-cert-manager-operator
  source: redhat-operators 
  sourceNamespace: openshift-marketplace
EOF
```
	
<!-- oc apply -f cert-manager-operatorgroup.yaml -->
<!-- oc apply -f cert-manager-subscription.yaml -->

- Confirm the subscription has been completed successfully before moving to the next step running the following command:
		
  		SUB_NAME=$(oc get deployment cert-manager-operator-controller-manager -n cert-manager-operator --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME -n cert-manager-operator --ignore-not-found -o jsonpath='{.status.phase}';fi;echo
	
  Wait Until You get a response like this:
		`Succeeded`

#### 2. Install Common Services Catalog Source

   - Deploy the Catalog Source

	oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-cp-common-services/4.6.17/OLM/catalog-sources.yaml

   Note: Reference for correct catalog sources for CP4I v16.1.0: [Catalog sources for operators](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=images-adding-catalog-sources-openshift-cluster#catalog-sources-for-operators)

  - Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

		oc get catalogsources opencloud-operators -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo

    Wait Until You get a response like this:
		`READY`

#### 3. Create common-services namespace:

	oc create namespace ibm-common-services

#### 4. Install  common-services Operator:

  _Optional: Installing the operators by using the Red Hat OpenShift console_

  - Create a Subscription for the IBM Cloud Pak foundational services operator using the example file. Save the file as common-service-subscription.yaml

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-common-service-operator
  namespace: openshift-operators
spec:
  channel: v4.6
  installPlanApproval: Automatic
  name: ibm-common-service-operator
  source: opencloud-operators
  sourceNamespace: openshift-marketplace
EOF
```

<!-- oc apply -f common-service-subscription.yaml -n openshift-operators -->

   - Confirm the operator has been deployed successfully before moving to the next step running the following command:
		
  	SUB_NAME=$(oc get deployment/ibm-common-service-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo
   
   Wait Until You get a response like this: 
   `Succeeded`
</details>


### Deploy Platform Navigator
<details closed>
Deploying the Platform UI allows you to deploy and manage instances from a central location.

1. Install Platform UI Catalog Source
   _Note: Reference for correct catalog sources for CP4I v16.1.0: Catalog sources for operators_

		oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-integration-platform-navigator/7.3.16/OLM/catalog-sources.yaml

   Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

		oc get catalogsources ibm-integration-platform-navigator-catalog -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
  
   Wait Until You get a response like this:
		`READY`

2.	Install Operator:
   
a.	Create a Subscription for the IBM Cloud Pak foundational services operator

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-integration-platform-navigator
  namespace: openshift-operators
spec:
  channel: v7.3-sc2
  name: ibm-integration-platform-navigator
  source: ibm-integration-platform-navigator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

<!-- oc apply -f platform-navigator-subscription.yaml -n openshift-operators -->

b.	Confirm the operator has been deployed successfully before moving to the next step running the following command:

	SUB_NAME=$(oc get deployment ibm-integration-platform-navigator-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo
 
   Wait Until You get a response like this(after few minutes):
	  `Succeeded`
	_Note: You may be seeing a response of PENDING which indicates the deployment is underway butnot yet complete. Wait until the READY response is received before continuing._
  
3.	Deploy the Platform UI instance
   
a.	Create Platform UI namespace and add pull secret to Namespace

	oc new-project cp4i

	oc create secret docker-registry ibm-entitlement-key   --docker-username=cp    --docker-password=$ENT_KEY  --docker-server=cp.icr.io     --namespace=tools
 
   __Note: The IBM Entitled Registry contains software images for the instances in IBM Cloud Pak® for Integration. To allow the operators to automatically pull those software images, you must first obtain your entitlement key, then add your entitlement key in a pull secret. Your entitlement key must be added to the OpenShift cluster as a pull secret to deploy instances. Adding a global pull secret enables deployment of instances in all namespaces. The alternative is to add a pull secret to each namespace in which you plan to deploy instances (any namespace with operators), plus the 'openshift-operators' namespace. However, this option adds work to your installation process._


b.	Create a PlatformNavigator with the following configuration. 

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: integration.ibm.com/v1beta1
kind: PlatformNavigator
metadata:
  name: cp4i-navigator
  namespace: cp4i
spec:
  integrationAssistant:
    enabled: true
  license:
    accept: true
    license: L-JTPV-KYG8TF
  replicas: 3
  version: 16.1.0
EOF
```

c.	Check the status of the Platform UI instance by running the following command in the project (namespace) where it was deployed:

	oc get platformnavigator cp4i-navigator -n cp4i -o jsonpath='{.status.conditions[0].type}';echo

   Wait Until You get a response like this: (Note: This can take upto 15mins)
       `Ready`

d.	Once the Platform UI instance is up and running get the access info:

Execute the following commands to retrieve the CP4I_URL, USER and Password:

	echo "CP4I Platform UI URL: $(oc get platformnavigator cp4i-navigator -n cp4i -o jsonpath='{.status.endpoints[?(@.name=="navigator")].uri}')";
	echo "CP4I admin user: $(oc get secret integration-admin-initial-temporary-credentials -n ibm-common-services -o jsonpath={.data.username} | base64 -d)";
	echo "CP4I admin password: $(oc get secret integration-admin-initial-temporary-credentials -n ibm-common-services -o jsonpath={.data.password} | base64 -d)"

 _Note the password is temporary and you will be required to change it the first time you log into Platform UI._

4. Login to CP4I
   
Use the browser to login to the CP4I url and upon successfully reset of password, you should see the following screen
 
<img width="1917" height="636" alt="image" src="https://github.com/user-attachments/assets/d33a090f-5dac-41f8-94b1-413dfc3e712f" />
</details>

### Deploy Asset Repo (optional)

<details closed>

1. Install Asset Repo Catalog Source
   _Note: Reference for correct catalog sources for CP4I v16.1.0: Catalog sources for operators_

		oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-integration-asset-repository/1.7.13/OLM/catalog-sources-linux-amd64.yaml

   Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

		oc get catalogsources ibm-integration-asset-repository-catalog -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
  
   Wait Until You get a response like this:
		`READY`

2.	Install Operator:
   
a.	Create a Subscription for the Asset Repo operator
  
```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-integration-asset-repository
  namespace: openshift-operators
spec:
  channel: v1.7-sc2
  name: ibm-integration-asset-repository
  source: ibm-integration-asset-repository-catalog
  sourceNamespace: openshift-marketplace
EOF
```

b.	Confirm the operator has been deployed successfully before moving to the next step running the following command:

	SUB_NAME=$(oc get deployment ibm-integration-asset-repository-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo
 
   Wait Until You get a response like this(after few minutes):
	  `Succeeded`
   
	_Note: You may be seeing a response of PENDING which indicates the deployment is underway butnot yet complete. Wait until the READY response is received before continuing._

3.	Deploy the Asset Repo instance
   
a.	Create Asset Repo namespace and add pull secret to Namespace

	oc new-project cp4i

	oc create secret docker-registry ibm-entitlement-key   --docker-username=cp    --docker-password=$ENT_KEY  --docker-server=cp.icr.io     --namespace=cp4i
 
   __Note: The IBM Entitled Registry contains software images for the instances in IBM Cloud Pak® for Integration. To allow the operators to automatically pull those software images, you must first obtain your entitlement key, then add your entitlement key in a pull secret. Your entitlement key must be added to the OpenShift cluster as a pull secret to deploy instances. Adding a global pull secret enables deployment of instances in all namespaces. The alternative is to add a pull secret to each namespace in which you plan to deploy instances (any namespace with operators), plus the 'openshift-operators' namespace. However, this option adds work to your installation process._


b.	Create a Asset Repo instance with the following configuration. 
    Set the correct storage file; In this case; 
  	For OCP_TYPE=ODF; we are setting OCP_FILE_STORAGE=`ocs-storagecluster-ceph-rbd` as seen in the YAML below.

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: integration.ibm.com/v1beta1
kind: AssetRepository
metadata:
  labels:
    backup.integration.ibm.com/component: assetrepository
  name: asset-repo-ai
  namespace: cp4i
spec:
  designerAIFeatures:
    enabled: true
  license:
    accept: true
    license: L-JTPV-KYG8TF
  replicas: 1
  singleReplicaOnly: true
  storage:
    assetDataVolume:
      class: ocs-storagecluster-ceph-rbd
    couchVolume:
      class: ocs-storagecluster-ceph-rbd
  version: 4.0-sc2
EOF
```
  <!-- oc apply -f platform-ui-instance.yaml  -n tools -->

c.	Check the status of the Asset Repo instance by running the following command in the project (namespace) where it was deployed:

	echo -n -e "\033[1;33m";oc get assetrepository asset-repo-ai -n cp4i -o jsonpath='{.status.phase}';echo -e "\033[0m"

   Wait Until You get a response like this: (Note: This can take upto 15mins)
       `Ready`

4. Post-deployment configuration (optional):
   
   a. Navigate to the Asset Repo instance from Platform UI clicking on the instance name as shown below:

   <img width="1843" height="586" alt="image" src="https://github.com/user-attachments/assets/3be099e9-d797-499e-9a7f-045d0345a1de" />

   b. From the main page select the Remotes tab and click Add Remote as shown below:
   <img width="1843" height="586" alt="image" src="https://github.com/user-attachments/assets/fb8f1218-4565-4e5d-a636-5491c6b4a3f8" />

   c. In the next page scroll all the way down and select Select All as shown below:
   <img width="1679" height="838" alt="image" src="https://github.com/user-attachments/assets/473d7e65-d2eb-40fd-a87e-17bd8f24fb00" />

   _Note at the moment not all the asset types are available in the repo but we are ready for future enhancements._
   
   d. Now scroll up again and enter the name of the remote repo, for instance `Joel CP4I Demo Assets` and then enter the Git URL "https://github.com/gomezrjo/cp4idemo" and then click "Create Remote" as shown below:
   <img width="1831" height="838" alt="image" src="https://github.com/user-attachments/assets/50c55ed5-e6b3-4a79-8c2b-3d2711bd0e55" />

   e. Final screen after adding new repo with assets
   <img width="1831" height="838" alt="image" src="https://github.com/user-attachments/assets/0f02836f-e235-4579-b078-a56e231b18c0" />

   You can add your own repo following the same process.

   
</details>

### Deploy Enterprise Messaging - MQ

<details closed>

1.	Install MQ Catalog Source:

   a. Deploy the Catalog source

	oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-mq/3.2.14/OLM/catalog-sources.yaml
 
   b. Confirm the catalog source has been deployed successfully before moving to the next step running the following command:
   
	oc get catalogsources ibmmq-operator-catalogsource -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
 
   Wait Until You get a response like this:
      `READY`
	  
2.	Install MQ Operator (2-5 mins):
   
   a. Create a Subscription for the MQ operator using the example file. Save the file as mq-subscription.yaml

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-mq
  namespace: openshift-operators
spec:
  channel: v3.2-sc2
  name: ibm-mq
  source: ibmmq-operator-catalogsource
  sourceNamespace: openshift-marketplace
EOF
```

<!-- oc apply -f mq-subscription.yaml -n openshift-operators --> 

  b. Confirm the operator has been deployed successfully before moving to the next step running the following command:
  
		SUB_NAME=$(oc get deployment ibm-mq-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo

  
   c. Wait Until You get a response like this:
      `Succeeded`

   
3.	Create MQ namespace and add pull secret to Namespace
   
		oc new-project cp4i-mq

		oc create secret docker-registry ibm-entitlement-key   --docker-username=cp    --docker-password=$ENT_KEY  --docker-server=cp.icr.io     --namespace=cp4i-mq

4.	Deploy Queue Manager Instance

Note: This is sample configuration for single Instance Queue Manager using MQSC and INI files. Additional configuration steps will be needed for more advanced MQ configuration and Security. 
	Creating a self-signed PKI using OpenSSL
	Example: Configuring a queue manager with mutual TLS authentication
	Testing a mutual TLS connection to a queue manager from your laptop
	Configuring high availability for queue managers using the IBM MQ Operator
	Configuring a Route to connect to a queue manager from outside a Red Hat OpenShift cluster

   a. Create mqsc-ini-example.yaml with the following:

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: mqsc-ini-example
  namespace: cp4i-mq
data:
  example1.mqsc: |
    DEFINE QLOCAL('DEV.QUEUE.1') REPLACE
    DEFINE QLOCAL('DEV.QUEUE.2') REPLACE    
  example2.mqsc: |
    DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
  example.ini: |
    Service:
      Name=AuthorizationService
      EntryPoints=14
      SecurityPolicy=UserExternal
EOF
```

  b. Create qmgr-demo-config.yaml with the following:
  
_(This yaml can also be generated via the platform navigator UI) 
(Navigate to Platform UI  Click Create Instance  Pick Queue Manager  Click next  Pick QuickStart configuration  Click Next  Toggle Advance Setting toggle switch  Enter the details  Click YAML ) Either copy+paste the new YAML or continue deploying MQ instance via UI)
_

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: mq.ibm.com/v1beta1
kind: QueueManager
metadata:
  name: qmgr-demo
  namespace: cp4i-mq
spec:
  version: 9.4.0.12-r1 # The identifier of the license you are accepting. This must be the correct license identifier for the version of MQ you are using. See https://ibm.biz/Bdm9be for valid values.
  license:
    accept: true
    license: L-JTPV-KYG8TF
    use: NonProduction
  queueManager:
    name: QMGRDEMO
    availability:
      type: SingleInstance
    mqsc:
    - configMap:
        name: mqsc-ini-example
        items:
        - example1.mqsc
        - example2.mqsc
    ini:
    - configMap:
        name: mqsc-ini-example
        items:
        - example.ini
    storage:
      defaultClass: ocs-storagecluster-ceph-rbd
      queueManager:
        type: persistent-claim
  web:
    console:
      authentication:
        provider: integration-keycloak
      authorization:
        provider: integration-keycloak
    enabled: true
EOF
```

<!-- oc apply -f mqsc-ini-example.yaml -n cp4i-mq --> 
<!-- oc apply -f qmgr-demo-config.yaml -n cp4i-mq --> 

  c. Confirm the instance has been deployed successfully before moving to the next step running the following command:
  
		oc get queuemanager qmgr-demo -n cp4i-mq -o jsonpath='{.status.phase}';echo
  
  d. Wait Until You get a response like this:
      `Running`
	  
  e. Execute the following command to verify that QMGR is running

		oc exec qmgr-demo-ibm-mq-0 -n cp4i-mq -- dspmq

5.	In the platform Navigator, you will now see any instance of Queue Manager running. 

   <img width="1917" height="636" alt="image" src="https://github.com/user-attachments/assets/44928bac-a75c-49f7-b5bf-7f54283ec1e1" />

   Click on the qmgr-demo link to navigate to Queue Manager console.
   
   <img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/17b8d24a-61bf-4b18-8a48-ccde6e783746" />


</details>

### Deploy App Connect
<details closed>

1.	Install App Connect Catalog Source:
    a. Apply the catalog source
	
 		oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-appconnect/12.0.15/OLM/catalog-sources.yaml

    b. Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

		oc get catalogsources appconnect-operator-catalogsource -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
 
   c. Wait Until You get a response like this:
		`READY`

2.	Install App Connect Operator: (Time Install ~2 mins)

   a. Create app-connect-subscription.yaml

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-appconnect
  namespace: openshift-operators     
spec:
  channel: v12.0-sc2
  name: ibm-appconnect
  source: appconnect-operator-catalogsource
  sourceNamespace: openshift-marketplace
EOF
```

<!-- oc apply -f app-connect-subscription.yaml -n openshift-operators -->

 b.	Confirm the operator has been deployed successfully before moving to the next step running the following command:

	SUB_NAME=$(oc get deployment ibm-appconnect-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo

 c. Wait Until You get a response like this:
	`Succeeded`

3.	Create new namespace and add entitlement key as secret

		oc new-project cp4i-ace

		oc create secret docker-registry ibm-entitlement-key   --docker-username=cp    --docker-password=$ENT_KEY  --docker-server=cp.icr.io     --namespace=cp4i-ace

4.	Deploy Dashboard instance:
   
    a. Create ace-dashboard-instance 

	   Set the correct storage file; In this case; 
  	   For OCP_TYPE=ODF; we are setting OCP_FILE_STORAGE=`ocs-storagecluster-cephfs` as seen in the YAML below.

       _(This yaml can also be generated via the platform navigator UI)
       (Navigate to Platform UI  Click Create Instance  Pick Integration Dashboard  Click next  Pick QuickStart configuration  Click Next  Toggle Advance Setting toggle switch  Enter the details  Click YAML ) Either copy+paste the new YAML or continue deploying instance via UI)_

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: appconnect.ibm.com/v1beta1
kind: Dashboard
metadata:
  labels:
    backup.appconnect.ibm.com/component: dashboard
  name: ace-dashboard
  namespace: cp4i-ace
spec:
  authentication:
    integrationKeycloak:
      enabled: true
  authorization:
    integrationKeycloak:
      enabled: true
  displayMode: IntegrationRuntimes
  license:
    accept: true
    license: L-XRNH-47FJAW
    use: CloudPakForIntegrationNonProduction
  pod:
    containers:
      content-server:
        resources:
          limits:
            memory: 512Mi
          requests:
            cpu: 50m
            memory: 50Mi
      control-ui:
        resources:
          limits:
            memory: 512Mi
          requests:
            cpu: 50m
            memory: 125Mi
  replicas: 1
  storage:
    size: 5Gi
    type: persistent-claim
    class: ocs-storagecluster-cephfs
  version: '12.0'
  api:
    enabled: true
EOF
```

<!-- oc apply -f  ace-dashboard-instance.yaml -n cp4i-ace -->

  b. Confirm the instance has been deployed successfully before moving to the next step running the following command:

		oc get dashboard ace-dashboard -n cp4i-ace -o jsonpath='{.status.phase}';echo
 
  c. Wait for few minutes Until You get a response like this:
  	`Ready`
  
  d. You should now see the ace-dashboard instance in the Platform Navigator UI

<img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/e3352cba-1cca-4987-a3ff-d51a8c3722c5" />

   Click on the ace-dashboard link to navigate to ACE 
   
<img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/e263fe9e-318f-4da3-8d43-3ed92b015ca1" />



5.	Deploy Designer Authoring instance 

    a. Create ACE designer instance

    _(This yaml can also be generated via the platform navigator UI)_
    _(Navigate to Platform UI  Click Create Instance  Pick Integration Design  Click next  Pick QuickStart with AI Enabled configuration  Click Next  Toggle Advance Setting toggle switch  Enter the details  Click YAML ) Either copy+paste the new YAML or continue deploying instance via UI)_

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: appconnect.ibm.com/v1beta1
kind: DesignerAuthoring
metadata:
  labels:
    backup.appconnect.ibm.com/component: designerauthoring
  name: ace-designer-ai
  namespace: cp4i-ace
spec:
  authentication:
    integrationKeycloak:
      enabled: true
  authorization:
    integrationKeycloak:
      enabled: true
  couchdb:
    replicas: 1
    storage:
      size: 10Gi
      type: persistent-claim
      class: ocs-storagecluster-ceph-rbd
  designerFlowsOperationMode: local
  designerMappingAssist:
    enabled: true
    incrementalLearning:
      schedule: Every 15 days
      useIncrementalLearning: true
      storage:
        type: persistent-claim
        class: ocs-storagecluster-cephfs
  license:
    accept: true
    license: L-XRNH-47FJAW
    use: CloudPakForIntegrationNonProduction
  replicas: 1
  version: '12.0'
EOF
```

<!-- oc apply -f ace-designer-local-ai-instance.yaml -n cp4i-ace -->

   b. Confirm the instance has been deployed successfully before moving to the next step running the following command:

   	oc get designerauthoring ace-designer-ai -n cp4i-ace -o jsonpath='{.status.phase}';echo

   c. Wait Until You get a response like this:
	`Ready`
 
   d. Once deployed, you should see the ace-designer instance in the platform navigator ui

<img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/5b5b9234-b43d-48a4-8529-a92199bcfdea" />

Click on the ace-designer-ai instance to launch the ACE Designer

<img width="1917" height="806" alt="image" src="https://github.com/user-attachments/assets/e1586967-d0fe-4a99-99b3-757a1c06f9dc" />


6.	Deploy Integration runtime instance 

    a. Create Integration runtime instance

    _(This yaml can also be generated via the platform navigator UI)_
    _(Navigate to Platform UI  Click Create Instance  Pick Integration Runtime  Click next  Pick QuickStart integration   Click Next  Toggle Advance Setting toggle switch  Enter the details  Click YAML ) Either copy+paste the new YAML or continue deploying instance via UI)_

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: appconnect.ibm.com/v1beta1
kind: IntegrationRuntime
metadata:
  labels:
    backup.appconnect.ibm.com/component: integrationruntime
  name: ace-runtime
  namespace: cp4i-ace
spec:
  license:
    accept: true
    license: L-XRNH-47FJAW
    use: CloudPakForIntegrationNonProductionFREE
  replicas: 1
  template:
    spec:
      containers:
        - name: runtime
          resources:
            requests:
              cpu: 300m
              memory: 368Mi
  version: '12.0'
EOF
```

   b. Confirm the instance has been deployed successfully before moving to the next step running the following command:

   	oc get integrationruntimes -n cp4i-ace -o=custom-columns='NAME:.metadata.name,STATUS:.status.phase' --sort-by=.metadata.name

   c. Wait Until You get a response like this:
	`Ready`
 
   d. Once deployed, you should see the ace-runtime instance in the platform navigator ui

   <img width="1783" height="747" alt="image" src="https://github.com/user-attachments/assets/a68a6b09-9169-45a2-bbbb-0a59250d2acd" />

   e. Click on the ace-runtime instance to launch the Integration runtime UI

   <img width="1783" height="747" alt="image" src="https://github.com/user-attachments/assets/c9aee348-019c-4e78-8c78-f5a52821802d" />


</details>

### Deploy APIC

<details closed>

1.	Install DataPower Catalog Source:

   a. Deploy the Catalog source

	oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-datapower-operator/1.11.7/OLM/catalog-sources.yaml
 
   b. Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

	oc get catalogsources ibm-datapower-operator-catalog -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
	
   Wait Until You get a response like this:
      `READY`
	  
2.	Install DataPower Operator:
   
   a. Create a Subscription for the DP operator using the example file.

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: datapower-operator
  namespace: openshift-operators      
spec:
  channel: v1.11-sc2
  name: datapower-operator
  source: ibm-datapower-operator-catalog
  sourceNamespace: openshift-marketplace
EOF
```

  b. Confirm the operator has been deployed successfully before moving to the next step running the following command:
  
		SUB_NAME=$(oc get deployment datapower-operator -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo 

   c. Wait Until You get a response like this:
      `Succeeded`

3.	Install APIC Catalog Source:

   a. Deploy the Catalog source

	oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-apiconnect/5.5.0/OLM/catalog-sources.yaml
 
   b. Confirm the catalog source has been deployed successfully before moving to the next step running the following command:

	oc get catalogsources ibm-apiconnect-catalog -n openshift-marketplace -o jsonpath='{.status.connectionState.lastObservedState}';echo
	
   Wait Until You get a response like this:
      `READY`
	  
3.	Install APIC Operator:
   
   a. Create a Subscription for the APIC operator using the example file.

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-apiconnect
  namespace: openshift-operators
spec:
  channel: v5.5-sc2
  name: ibm-apiconnect
  source: ibm-apiconnect-catalog
  sourceNamespace: openshift-marketplace
EOF
```

  b. Confirm the operator has been deployed successfully before moving to the next step running the following command:
  
		SUB_NAME=$(oc get deployment ibm-apiconnect -n openshift-operators --ignore-not-found -o jsonpath='{.metadata.labels.olm\.owner}');if [ ! -z "$SUB_NAME" ]; then oc get csv/$SUB_NAME --ignore-not-found -o jsonpath='{.status.phase}';fi;echo    

   c. Wait Until You get a response like this:
      `Succeeded`


4.	Create APIC namespace and add pull secret to Namespace
   
		oc new-project cp4i-apic

		oc create secret docker-registry ibm-entitlement-key   --docker-username=cp    --docker-password=$ENT_KEY  --docker-server=cp.icr.io     --namespace=cp4i-apic

5.	Deploy APIC Instance

  a. Create apic-demo-config.yaml with the following:
  
_(This yaml can also be generated via the platform navigator UI) 
(Navigate to Platform UI  Click Create Instance  Pick Queue Manager  Click next  Pick QuickStart configuration  Click Next  Toggle Advance Setting toggle switch  Enter the details  Click YAML ) Either copy+paste the new YAML or continue deploying MQ instance via UI)
_

```yaml annotate
cat <<EOF | oc apply -f -

EOF
```

  c. Confirm the instance has been deployed successfully before moving to the next step running the following command:
  
		
  
  d. Wait Until You get a response like this:
      `Running`

6.	In the platform Navigator, you will now see an instance of APIC running. 



   Click on the apic-demo instance link to navigate to APIC console.
   


</details>


## Additional References

### Structuring CP4I Deployments in Namespaces 

Resources:
- https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.2?topic=ui-managing-instances-across-multiple-locations
- https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.2?topic=planning-structuring-your-deployment
- https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=installing-operators


### Catalog Sources
<details closed>
Add a separate catalog source for each operator in your OpenShift cluster, to make the IBM operators available for installation. This task is also required to apply the fix packs for catalog sources, prior to an upgrade. Using a separate catalog source for each operator gives you full control of software versioning on an OpenShift cluster. It enables the following:
•	Upgrade each Cloud Pak component independently.
•	Have a fully declarative set of artifacts, which you can use to recreate exact installations.
•	Easily control upgrade and promotion through environments (such as from test to production environments) with a CI/CD pipeline.
•	Control when upgrades happen. A new operator version becomes available in an OpenShift cluster only after you update the catalog source for that operator. This process effectively gives you manual control of upgrades, without actually using the Manual option (which is not recommended; see "Before you begin" for more information).

CATALOG SOURCE YAML
Reference: https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=images-adding-catalog-sources-openshift-cluster#catalog-sources-for-operators__title__1

| Name of Service      | Command reference       |
| -------------- | -------------- |
| IBM Cloud Pak foundational services | ```oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-cp-common-services/4.6.17/OLM/catalog-sources.yaml``` |
| IBM Cloud Pak for Integration | ```oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-integration-platform-navigator/7.3.16/OLM/catalog-sources.yaml``` |
| IBM Automation foundation assets | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-integration-asset-repository/1.7.13/OLM/catalog-sources-linux-amd64.yaml ``` \
| IBM API Connect | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-apiconnect/5.5.0/OLM/catalog-sources.yaml ``` |
| IBM App Connect | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-appconnect/12.0.15/OLM/catalog-sources.yaml ``` |
| IBM MQ | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-mq/3.2.14/OLM/catalog-sources.yaml ``` |
| IBM DataPower Gateway | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-datapower-operator/1.11.7/OLM/catalog-sources.yaml ``` |
| IBM Event Streams | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-eventstreams/12.0.0/OLM/catalog-sources.yaml ``` |
| IBM Event Endpoint Management | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-eventendpointmanagement/11.6.3/OLM/catalog-sources.yaml ``` |
| IBM Event Processing | ``` oc apply --filename https://raw.githubusercontent.com/IBM/cloud-pak/master/repo/case/ibm-eventprocessing/1.4.2/OLM/catalog-sources.yaml ``` |

</details>

## Subscription Files 
<details closed>
Subscription YAML 

Here are the Subscription YAML files used for building the env, which is based upon CP4I 16.1.0
For a full list of subscriptions, see [Operators available to install.](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=operators-installing-by-using-cli#operators-available)



- IBM Cloud Pak for Integration - Platform UI, Assembly, API, API Product, Messaging server, Messaging channel, Messaging queue, Messaging user
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-integration-platform-navigator
  namespace: openshift-operators
  labels:
    backup.integration.ibm.com/component: subscription        
spec:
  channel: v7.3-sc2
  name: ibm-integration-platform-navigator
  source: ibm-integration-platform-navigator-catalog
  sourceNamespace: openshift-marketplace
```

- IBM Cloud Pak foundational services - Cloud Pak foundational services for Cloud Native PostgreSQL and RedHat Build of Keycloak only
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-common-service-operator
  namespace: openshift-operators
  labels:
    backup.integration.ibm.com/component: subscription        
spec:
  channel: v4.6
  name: ibm-common-service-operator
  source: opencloud-operators
  sourceNamespace: openshift-marketplace
```

- IBM Automation foundation assets - Automation assets
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-integration-asset-repository
  labels:
    backup.integration.ibm.com/component: subscription        
spec:
  channel: v1.7-sc2
  name: ibm-integration-asset-repository
  source: ibm-integration-asset-repository-catalog
  sourceNamespace: openshift-marketplace
```

- IBM API Connect - API Connect cluster, API Manager, API Analytics, API Portal, API Gateway
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-apiconnect
  labels:
    backup.apiconnect.ibm.com/component: subscription        
spec:
  channel: v5.5-sc2
  name: ibm-apiconnect
  source: ibm-apiconnect-catalog
  sourceNamespace: openshift-marketplace
```
	
- IBM App Connect - Integration dashboard, Integration design, Integration runtime
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-appconnect
  labels:
    backup.appconnect.ibm.com/component: subscription        
spec:
  channel: v12.0-sc2
  name: ibm-appconnect
  source: appconnect-operator-catalogsource
  sourceNamespace: openshift-marketplace
```

- IBM MQ - Queue manager
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-mq
  labels:
    backup.mq.ibm.com/component: subscription        
spec:
  channel: v3.2-sc2
  name: ibm-mq
  source: ibmmq-operator-catalogsource
  sourceNamespace: openshift-marketplace
```

- IBM Event Streams - Kafka cluster, Kafka topic, Kafka user, Kafka Connect runtime, Kafka connector
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-eventstreams
  labels:
    backup.eventstreams.ibm.com/component: subscription        
spec:
  channel: v12.0
  name: ibm-eventstreams
  source: ibm-eventstreams
  sourceNamespace: openshift-marketplace
```

- IBM Event Endpoint Management - Event Manager, Event Gateway
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-eventendpointmanagement
  labels:
    backup.events.ibm.com/component: subscription        
spec:
  channel: v11.6
  name: ibm-eventendpointmanagement
  source: ibm-eventendpointmanagement-catalog
  sourceNamespace: openshift-marketplace
```

- IBM Event Processing - Event Processing
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ibm-eventprocessing
spec:
  channel: v1.4
  name: ibm-eventprocessing
  source: ibm-eventprocessing-catalog
  sourceNamespace: openshift-marketplace
```

- IBM DataPower Gateway - Enterprise gateway
  _Important: Do not apply this subscription if you already applied the subscription for the IBM API Connect operator
``` yaml annotate
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: datapower-operator
  labels:
    backup.datapower.ibm.com/component: subscription        
spec:
  channel: v1.11-sc2
  name: datapower-operator
  source: ibm-datapower-operator-catalog
  sourceNamespace: openshift-marketplace
```

</details>

