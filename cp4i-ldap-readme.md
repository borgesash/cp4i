# CP4I LDAP integration 
*This document is intended to capture the steps of integrating LDAP users to manage CP4I.*


Refer to following link for detailed information: [Configuring an LDAP identity provider](https://www.ibm.com/docs/en/cloud-paks/cp-integration/16.1.0?topic=management-managing-users-groups)


## Configuring LDAP for Authentication

This assumes that you are going to import all users into keyCloak. If you only want to import users belonging to an LDAP group, then skip this section and follow instructions in the section [Configuring LDAP Groups]()

Login to Keycloak UI using `integration-admin` user (Login to Platform UI --> Click Access Control. See screen below)

<img width="300" height="413" alt="IBM Cloud Pak for Integration" src="https://github.com/user-attachments/assets/b95e569d-11c1-47e9-a43f-28b0d2eb62c1" />


Click User Federation and and select ‘Add Ldap providers’. 

<img width="1262" height="435" alt="Pasted Graphic 15" src="https://github.com/user-attachments/assets/77c4b4f7-7cb4-4d3d-9b6d-830eb4b14472" />

Here's the initial screen showing only the default user `integration-admin `

<img width="1855" height="472" alt="Pasted Graphic 1" src="https://github.com/user-attachments/assets/aa6e0390-fa3e-42b6-8c08-2634bf7d97c1" />

Enter your LDAP details.
For purpose of this demo, I am using `jumpcloud Ldap` as my LDAP, so below screens are taken while configuring `jumpcloud Ldap` as my IdP

<img width="1829" height="714" alt="image" src="https://github.com/user-attachments/assets/e7ecae0e-27e9-4ee8-a320-92edb163cec4" />
￼
<img width="1829" height="714" alt="Pasted Graphic 90" src="https://github.com/user-attachments/assets/0cf3b83d-2b21-417b-b585-f05890cb43d4" />

<img width="1862" height="861" alt="Pasted Graphic 59" src="https://github.com/user-attachments/assets/631eb099-043b-4cdd-b8b6-468f03390009" />

<img width="1862" height="698" alt="Pasted Graphic 57" src="https://github.com/user-attachments/assets/b2adbfc0-0703-4c48-8b7e-3b82ac3ce9d6" />

<img width="1862" height="861" alt="Pasted Graphic 58" src="https://github.com/user-attachments/assets/5c9b9804-66bc-48c5-9cba-29e7fe6654fa" />

Ensure that `Test connection` and `Test authentication` tests are successful before proceeding to next steps.
In the LDAP Searching and updating section, ensure that you have correct attributes mapped. 

In case any LDAP searching filters are incorrect, you will see error when you navigate to Users 

<img width="1857" height="788" alt="Pasted Graphic 23" src="https://github.com/user-attachments/assets/2559f1b1-da73-4fe6-83a7-a77585bcd998" />

For LDAP server used for this demo, the UUID LDAP attribute was incorrect, so after changing it to “uuid”, the issue was resolved 

<img width="1857" height="788" alt="Pasted Graphic 24" src="https://github.com/user-attachments/assets/bc902236-f6eb-4b7a-abdc-95b499cdd4e8" />


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


## Configuring LDAP Groups

The below instructions are for a scenario where you want members of group  `CP4I Admins` to get access to CP4I 

<img width="655" height="187" alt="• (P4I Admins, Users, 680d33653161430ef15SFfSA," src="https://github.com/user-attachments/assets/4ef30489-05ea-49a0-ae8f-2a2d69bfee57" />

Here's the initial screen showing `No Groups`

<img width="1869" height="472" alt="Pasted Graphic" src="https://github.com/user-attachments/assets/b63c2112-4f43-42b4-ac4c-54c020ef0b09" />

Click User Federation and and select `Add Ldap providers`. Enter your LDAP details.

<img width="1262" height="435" alt="Pasted Graphic 15" src="https://github.com/user-attachments/assets/77c4b4f7-7cb4-4d3d-9b6d-830eb4b14472" />

Enter your LDAP details.
For purpose of this demo, I am using `jumpcloud Ldap` as my LDAP, so below screens are taken while configuring `jumpcloud Ldap` as my IdP

Enter your LDAP configuration and click `Test Connection` to verify LDAP url is correct. Click `test authentication` to validate the credentials. 

<img width="1868" height="800" alt="Pasted Graphic 2" src="https://github.com/user-attachments/assets/701bf0e5-5e8e-4acf-8b78-ebb28278df3a" />

<img width="1851" height="812" alt="Pasted Graphic 16" src="https://github.com/user-attachments/assets/7f803bf0-847f-4961-9dbf-b86f566925c0" />

Ensure that both tests are successful before proceeding to next steps.

In the LDAP Searching and updating section, ensure that you have correct attributes mapped. 

Below configuration is based upon following LDAP users 

<img width="772" height="383" alt="Pasted Graphic 21" src="https://github.com/user-attachments/assets/d8d197bc-3f05-4763-bfb6-56dc61ab1c80" />

<img width="1868" height="685" alt="Pasted Graphic 3" src="https://github.com/user-attachments/assets/47f3ea98-3c44-4e63-b3b9-4412d78d3514" />

<img width="1857" height="788" alt="Pasted Graphic 22" src="https://github.com/user-attachments/assets/2fb139cd-b58f-4f5e-9c8b-1c57ecdd1f6a" />

Sync Settings 

Turn off everything since we don’t want to import any users 
ALSO, Make sure to enter a value in the `User LDAP Filter` that DOES NOT IMPORT any users, because keycloak will import users even though 'import users' option is disabled.

<img width="1851" height="589" alt="Pasted Graphic 20" src="https://github.com/user-attachments/assets/1e3e8f32-0fd4-4d88-a085-6497cd7cf721" />

<img width="1851" height="692" alt="Pasted Graphic 17" src="https://github.com/user-attachments/assets/772de197-ab25-4978-b13f-5001eb79fd63" />

<img width="1851" height="589" alt="Pasted Graphic 18" src="https://github.com/user-attachments/assets/30039b9e-8497-4cea-8089-7ff913f37e9f" />


Click SAVE

<img width="1851" height="589" alt="Pasted Graphic 19" src="https://github.com/user-attachments/assets/7de99859-4eaf-48f5-b0d4-1e51004c5a17" />


### CREATE A MAPPER

The below LDAP group  `CP4I Admins` was used to configure the group-mapper 

<img width="655" height="187" alt="• (P4I Admins, Users, 680d33653161430ef15SFfSA," src="https://github.com/user-attachments/assets/4ef30489-05ea-49a0-ae8f-2a2d69bfee57" />

<img width="1868" height="797" alt="Pasted Graphic 7" src="https://github.com/user-attachments/assets/af33eb96-45a0-41ba-9e7f-bc2e04fe52a4" />

Update user-attribute mapper - _This is to ensure that correct LDAP attribute(uid or dn or cn) is mapped to the userId used to login to CP4I_

<img width="965" height="680" alt="Pasted Graphic 9" src="https://github.com/user-attachments/assets/ae7a9875-e664-44aa-87cf-0d775b70def8" />


Navigate to the new group mapper and click Action — > Sync LDAP groups to LDAP

<img width="1867" height="590" alt="Pasted Graphic 10" src="https://github.com/user-attachments/assets/53e8c352-373e-46f1-801c-9c839d6523db" />

<img width="1867" height="590" alt="Pasted Graphic 11" src="https://github.com/user-attachments/assets/f3a10595-30c9-4c76-abf6-ffdcbd8b2551" />

If your filters are correct you will see the message

<img width="875" height="204" alt="Data successfully synced 0 imported groups, 1 updated groups, 0 removed" src="https://github.com/user-attachments/assets/c4ed7361-1a06-4738-8929-db439a8b1895" />

If Filters are Incorrect, you will see following message:

<img width="875" height="204" alt="Pasted Graphic 14" src="https://github.com/user-attachments/assets/ad20706e-589e-4eec-b88f-4e0f2bb18833" />



## Additional resources
- [Troubleshooting LDAP configuration](https://www.ibm.com/docs/en/cloud-private/3.2.x?topic=ldap-troubleshooting-configuration)


---
