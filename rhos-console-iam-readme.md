# Red Hat OpenShift Console LDAP integration 
*This document is intended to capture the steps of integrating LDAP users to manage Red Hat OpenShift.*

By default, only a kubeadmin user exists on your cluster. To specify an identity provider, you must create a custom resource (CR) that describes that identity provider and add it to the cluster. 

Refer to following link for detailed information: [Configuring an LDAP identity provider](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/authentication_and_authorization/configuring-identity-providers#configuring-ldap-identity-provider)

Upon initial install of OCP, the following message is displayed when logging into Red Hat Console
`You are logged in as a temporary administrative user. Update the cluster OAuth configuration to allow others to log in.`

Either Click on the "update the cluster OAuth" link in message to navigate to Cluster OAuth screen where you can add `LDAP` as the Identity provider via the UI or follow the steps below using CLI.


## Configuring LDAP for Authentication

You can verify the OAuth Provider with the following check:

```bash
oc get oauth cluster -o yaml
```

If using the default kubeadmin, you should see output similar to 

```yaml annotate
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - mappingMethod: claim
    name: IBMid
    openID:
      claims:
        email:
        - email
        groups:
        - member_of
        preferredUsername:
        - email
      clientID: d74c0bb8-60a7-482f-9520-6a9a7a3d153e
      clientSecret:
        name: openid-client-secret-isv
      extraScopes: []
      issuer: https://techzone.verify.ibm.com/oidc/endpoint/default
    type: OpenID

```
Now, lets proceed with the steps below:

1. Create Secret object that contains the bindPassword field

```bash
oc create secret generic ldap-secret --from-literal=bindPassword=<secret> -n openshift-config
```

2. Create a ConfigMap object in `openshift-config` namespace containing the certificate authority by using the following command 

```bash
oc create configmap ca-config-map --from-file=ca.crt=/path/to/ca-file -n openshift-config
```

3. Create CR for LDAP as Identity Provider

```yaml annotate
cat <<EOF > ldap-cr.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ldapidp 
    mappingMethod: claim 
    type: LDAP
    ldap:
      attributes:
        id: 
        - uid
        email: 
        - mail
        name: 
        - cn
        preferredUsername: 
        - uid
      bindDN: "<<enter bind dn>>" 
      bindPassword: 
        name: ldap-secret
      ca: 
        name: ca-config-map
      insecure: false # When false, ldaps URLs connect using TLS
      url: "ldaps://<<host:port>>/<<dn>>?uid"  #specifies the LDAP host and search parameters to use
EOF
```

Edit the ldap-cr.yaml file as needed and then Apply the CR using the following command
```bash
oc apply -f ldap-cr.yaml
```

You can verify the OAuth Provider afterwards with the following check:

```bash
oc get oauth cluster -o yaml
```

Sample output using `jumpcloud ldap`

```yaml annotate
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - ldap:
      attributes:
        email:
        - mail
        id:
        - uid
        name:
        - cn
        preferredUsername:
        - uid
      bindDN: uid=ocp-admin,ou=Users,o=68dd336531b443aef155ff54,dc=jumpcloud,dc=com
      bindPassword:
        name: ldap-secret
      ca:
        name: ca-config-map
      insecure: false
      url: ldaps://ldap.jumpcloud.com:636/ou=Users,o=68dd336531b443aef155ff54,dc=jumpcloud,dc=com?uid
    mappingMethod: claim
    name: ldapidp
    type: LDAP
```

4. Validation

Log in to the cluster as a user from your identity provider, entering the password when prompted. 

```
oc login -u <username>
```

Confirm that the user logged in successfully, and display the user name. 

```
oc whoami
```

5. Configuring Roles

Next we want to either grant permissions to a single user `devops-user1`  or to a group of users within `devops-team`:

```bash
oc adm policy add-cluster-role-to-user cluster-admin devops-user1
```

To add a group of users, you need to sync the LDAP Groups with RHOS.

First, create a groupsync.yaml file with your specific information in it:

```yaml
apiVersion: v1
kind: LDAPSyncConfig
url: "ldap://<<ldap-host-name>>:389"
bindDN: "uid=<<fully-qualified-name>>"
bindPassword:
  file: "/home/deploy/ldap-config/bindPassword"
insecure: true
rfc2307:
  groupsQuery:
    baseDN: "<<base-dn>>"
    scope: sub
    derefAliases: never
    filter: "(objectClass=groupOfNames)"
  groupUIDAttribute: "dn"
  groupNameAttributes:
    - cn
  groupMembershipAttributes:
    - member
  usersQuery:
    baseDN: "<<add-here>>"
    scope: sub
    derefAliases: never
  userUIDAttribute: "dn"
  userNameAttributes:
    - uid
groupUIDNameMapping:
  "cn=devops-team,....,dc=com": "devops-team" #this is the cn of the devops group
 ```

You need to create the `bindPassword` file before syncing the LDAP group:

```bash
echo 'your_ldapbind_password' | sudo tee /home/deploy/ldap-config/bindPassword > /dev/null
chmod 644 /home/deploy/ldap-config/bindPassword
tr -d '\n' < /home/deploy/ldap-config/bindPassword > temp && mv temp /home/deploy/ldap-config/bindPassword
```

 Once you have created the `groupsync.yaml` file, you can sync RHOS:

 ```bash
 oc adm groups sync \
  --sync-config=groupsync.yaml \
  --confirm
```

Verify the group was created:

```bash
oc get group devops-team -o yaml
```

Assign the group Cluster-wide Access:

```bash
oc adm policy add-cluster-role-to-group cluster-admin devops-team
```

Or assign the group Namespace-specific Access:

```bash
oc adm policy add-role-to-group edit devops-team -n devops-project
```

For more information on RBAC refer to [link](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/authentication_and_authorization/using-rbac)

---
