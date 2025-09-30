# Red Hat OpenShift IAM integration 
*This document is intended to capture the steps of integrating LDAP users to manage Red Hat OpenShift.*

## Configuring LDAP for Authentication

```yaml
identityProviders:
- name: ldap_provider
  mappingMethod: claim
  type: LDAP
  ldap:
    url: "ldap://<<add-here>>?uid"
    bindDN: "uid=<<fully-qualified-name>>"
    bindPassword:
      name: ldap-bind-secret #this should match the secret-name created in next step
    insecure: true
    attributes:
      id: ["uid"]
      email: ["mail"]
      name: ["cn"]
      preferredUsername: ["uid"]
```

Additional steps are required after adding the authentication source in the GUI or through the CLI.

```bash
oc create secret generic ldap-bind-secret \
  --namespace=openshift-config \
  --from-literal=bindPassword='<<password of user used as binDN>>'
```

You can verify the OAuth Provider afterwards with the following check:

```bash
oc get oauth cluster -o yaml
```

Next we want to either grant permissions to a single user or to a group of users:

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
