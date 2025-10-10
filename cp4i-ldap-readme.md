# CP4I LDAP integration 
*This document is intended to capture the steps of integrating LDAP users to manage CP4I.*


Refer to following link for detailed information: [Configuring an LDAP identity provider](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=management-managing-users-groups)


## Configuring LDAP for Authentication


Login to Keycloak UI using `integration-admin` user (Login to Platform UI --> Click Access Control. See screen below)

<img width="300" height="413" alt="IBM Cloud Pak for Integration" src="https://github.com/user-attachments/assets/b95e569d-11c1-47e9-a43f-28b0d2eb62c1" />


Click User Federation and enter your LDAP details.
For purpose of this demo, I am using `jumpcloud Ldap` as my LDAP, so below screens are taken while configuring `jumpcloud Ldap` as my IdP

<img width="1829" height="714" alt="image" src="https://github.com/user-attachments/assets/e7ecae0e-27e9-4ee8-a320-92edb163cec4" />
￼
<img width="1829" height="714" alt="Pasted Graphic 90" src="https://github.com/user-attachments/assets/0cf3b83d-2b21-417b-b585-f05890cb43d4" />

<img width="1862" height="861" alt="Pasted Graphic 59" src="https://github.com/user-attachments/assets/631eb099-043b-4cdd-b8b6-468f03390009" />

<img width="1862" height="698" alt="Pasted Graphic 57" src="https://github.com/user-attachments/assets/b2adbfc0-0703-4c48-8b7e-3b82ac3ce9d6" />

<img width="1862" height="861" alt="Pasted Graphic 58" src="https://github.com/user-attachments/assets/5c9b9804-66bc-48c5-9cba-29e7fe6654fa" />


UPON clicking SAVE , the following screen is displayed

<img width="1872" height="434" alt="Pasted Graphic 60" src="https://github.com/user-attachments/assets/2fdbf214-fccd-4a53-98bd-fee8031c012f" />


Click Users to see the users imported from LDAP

<img width="1872" height="434" alt="Pasted Graphic 61" src="https://github.com/user-attachments/assets/5151bba1-7590-4491-a1aa-8074b8a128e0" />

Use the `Mappers` TAB to fix any incorrectly mapped LDAP attributes 

<img width="1910" height="594" alt="image" src="https://github.com/user-attachments/assets/e2fdad5c-1feb-416a-88f8-196b6667629b" />

When Trying to login to CP4I UI Console using the LDAP userId, if you get the Access Denied 403 error, it means that there is NO ROLE assigned to new User.

<img width="1872" height="861" alt="Pasted Graphic 62" src="https://github.com/user-attachments/assets/2cc29b9f-8e56-4579-a5a2-6b8f2558b660" />


Navigate to keyCloak Console UI

Goto Users—> Click on user (in this case we are using cp4i-admin)  —> Click on ‘Role Mapping’ tab

<img width="1903" height="853" alt="Pasted Graphic 64" src="https://github.com/user-attachments/assets/a5f575e8-a8bc-48ea-8728-235765e78813" />


Click on Assign Role and pick  `integration-xxxx` - This will allow the new user full ADMIN access to CP4I UI similar to the original `integration-admin` user. 
For additional details on Roles and Permissions refer [link](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=management-cloud-pak-roles-permissions)

<img width="1903" height="853" alt="Pasted Graphic 65" src="https://github.com/user-attachments/assets/cbae228c-520b-4957-b359-04f7e80ca7cd" />


Click Assign 

<img width="1903" height="443" alt="Pasted Graphic 66" src="https://github.com/user-attachments/assets/fd26a7a6-eaa6-4803-b930-ed199f958f04" />


Now navigate to CP4I url and login as the `cp4i-admin` user 

<img width="1903" height="890" alt="Pasted Graphic 63" src="https://github.com/user-attachments/assets/cc9af771-d5cc-4eb3-b4db-491904f9a46c" />


## Adding certificates to the Keycloak trust store
You can add certificates to the Keycloak trust store so that Keycloak securely connects to services protected by a custom certificate authority.
To add certificates to the Keycloak trust store, you must be a user with namespace admin permissions. 

For more information, see [OpenShift roles and permissions](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=management-keycloak-configuration#adding-certificates-to-the-keycloak-trust-store)

- Identify the namespace that contains Keycloak.
```bash annotate
KEYCLOAK_NAMESPACE=<namespace>
```

For installations using All namespaces on the cluster mode, this is the servicesNamespace defined in the CommonService resource (which is `ibm-common-services` by default).

- Create the resource with your additional certificates.

If using Keycloak version 26, create a secret named cs-keycloak-ca-certs with your additional certificates. 

```oc get csv -A | grep -i keycloak```

Sample Output showing v26
<img width="1595" height="29" alt="image" src="https://github.com/user-attachments/assets/129e0cf0-2f79-4780-8588-7a161453ad9d" />


Use one key for each certificate:

```bash annotate
oc create secret generic cs-keycloak-ca-certs --from-file=cert1.pem --from-file=cert2.pem -n ${KEYCLOAK_NAMESPACE}
```

- To apply the configuration changes, restart the cs-keycloak pods:

```bash annotate
oc rollout restart statefulset/cs-keycloak -n ${KEYCLOAK_NAMESPACE}
```

## How to Enable DEBUG for KEY CLOAK


In RH Console, Navigate to Workloads —> StatefulSets —> find ‘cs-keycloak’ —> Click ‘Environment’ tab 

￼
<img width="1595" height="344" alt="Pasted Graphic 93" src="https://github.com/user-attachments/assets/f8db0885-b0f7-4c03-9bee-a04532c5fd9f" />


Click “Add More” to add a new ENV variable. 
Name: `KC_LOG_LEVEL` 
Value: `DEBUG`

<img width="1595" height="581" alt="Pasted Graphic 94" src="https://github.com/user-attachments/assets/39ae75dc-3ca3-4beb-b09c-6a9929059a57" />


Click SAVE , which will trigger the POD restart 

Click PODS TAB and confirm that Pod has restart by looking the Created Date

<img width="1854" height="581" alt="Pasted Graphic 95" src="https://github.com/user-attachments/assets/5b5e1663-64d7-407f-8864-d2ea3f3c9e8b" />


Click on the POD `cs-keycloak-0` and navigate to the Logs to confirm that you can see the DEBUG logs

CLI command to view the logs ```oc logs pod/cs-keycloak-0 -n ibm-common-services```

<img width="1854" height="581" alt="Pasted Graphic 96" src="https://github.com/user-attachments/assets/b72d1a71-91a3-4a22-bf5f-9406296efaa9" />



---
