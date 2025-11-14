# ACE BAR file deploying using CLI

This tutorial introduces some IBM® App Connect in containers concepts and describes how you can deploy a broker archive (BAR) file, which was developed by using the IBM App Connect Enterprise Toolkit, to a runtime environment.

## Steps

1. Log in to the Red Hat OpenShift cluster

`oc login --token=xxxxxxxxx  --server=https://host:port`

Create a new namespace

```yaml annotate
oc new-project cp4i-ace-demo
```

2. Creating a configuration object from the CLI

a. Prepare the authentication credentials for the GitHub location. You can specify the credentials in the following JSON format:

`{"authType":"BASIC_AUTH","credentials":{"username":"myUsername","password":"myPassword"}}`

Because the BAR file is in a public GitHub location, a username and password are not required. So you can simply specify the credentials by passing empty values:

b. Base64 encode the authentication credentials
   
```yaml annotate
echo '{"authType":"BASIC_AUTH","credentials":{"username":"","password":""}}' | base64
```

The Base64-encoded output is as follows:

`eyJhdXRoVHlwZSI6IkJBU0lDX0FVVEgiLCJjcmVkZW50aWFscyI6eyJ1c2VybmFtZSI6IiIsInBhc3N3b3JkIjoiIn19Cg==`


c. Run the following command to create the Configuration object:

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: appconnect.ibm.com/v1beta1
kind: Configuration
metadata:
  name: github-barauth
  namespace: cp4i-ace-demo
spec:
  data: eyJhdXRoVHlwZSI6IkJBU0lDX0FVVEgiLCJjcmVkZW50aWFscyI6eyJ1c2VybmFtZSI6IiIsInBhc3N3b3JkIjoiIn19Cg== #replace this value with the output of step b
  description: Authentication for GitHub
  type: barauth
EOF
```

The command should produce the following output:
`configuration.appconnect.ibm.com/github-barauth created`

You can check the status of your configuration object or list all the configuration objects that you have created by using following command:

```yaml annotate
oc get configuration -n cp4i-ace-demo
```

You should see output that is similar to this:

<img width="677" height="29" alt="image" src="https://github.com/user-attachments/assets/415bbe0f-6d4b-4d13-b486-f5da9df86a5b" />


3. Deploying the BAR file to an integration runtime by using the CLI

Create the integration runtime in the cluster by running the following command

```yaml annotate
cat <<EOF | oc apply -f -
apiVersion: appconnect.ibm.com/v1beta1
kind: IntegrationRuntime
metadata:
  name: http-echo-service
  namespace: cp4i-ace-demo
spec:
  license:
    accept: true
    license: L-KPRV-AUG9NC
    use: CloudPakForIntegrationNonProductionFREE
  template:
    spec:
      containers:
        - resources:
            requests:
              cpu: 300m
              memory: 368Mi
          name: runtime
  logFormat: basic
  barURL:
    - 'https://github.com/amarIBM/hello-world/raw/master/HttpEchoApp.bar'
  configurations:
    - github-barauth
  version: '13.0'
  replicas: 1
EOF
```

You should receive this confirmation:

`integrationruntime.appconnect.ibm.com/http-echo-service created`


4. Testing the message flow

a. Verify the status of the integration runtime pod.

```yaml annotate
oc get pods -n cp4i-ace-demo
```

You should see output that is similar to this:

<img width="677" height="29" alt="image" src="https://github.com/user-attachments/assets/9d5027ae-4588-44d8-8942-edd01d231580" />

b. You can also verify the status of your application by looking at the pod log:

```yaml annotate
oc logs <podName>
```

For example:

`oc logs http-echo-service-ir-66c689ddc-n7xvw`

You should see output that is similar to this:

From the pod logs, you can see that the deployed HTTPEcho service is listening on service endpoint /Echo.


c. Identify the external URL (that is, the public endpoint) for your service by using routes:

```yaml annotate
oc get routes
```

You should see output that is similar to this:

<img width="1309" height="44" alt="image" src="https://github.com/user-attachments/assets/4068ada6-35ef-4fc6-a34e-e7d24561e6e4" />

The HOST/PORT values represent the external base URL for invoking the service.


d. Invoke the service by using the curl command to call an endpoint that is constructed from the URL for the http-echo-service-http route from the previous step, and the service endpoint /Echo.

```yaml annotate
curl -X POST http://HOST/PORT_value/Echo
```

For example:

`curl -X POST http://http-echo-service-http-cp4i-ace-demo.apps.691542220bee05b668b6e3a3.am1.techzone.ibm.com/Echo`

You should receive a response that is similar to the following example, which confirms that your request was successful.

`<Echo><DateStamp>2025-11-13T17:27:05.220219Z</DateStamp></Echo>`


## Whats Next:

In this scenario, you deployed a simple flow from an IBM Integration Bus or IBM App Connect Enterprise environment into a container by using the Red Hat OpenShift CLI. You can now proceed to Scenario 2 to learn how to deploy the same BAR file by using the [App Connect Dashboard](https://github.com/borgesash/cp4i/blob/main/ace/deploy-sample-ace-bar-using-ui.md).


## References

https://www.ibm.com/docs/en/app-connect/13.0.x?topic=dtiir-scenario-1-deploying-toolkit-integration-from-red-hat-openshift-cli-1

