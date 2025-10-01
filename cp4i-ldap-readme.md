# CP4I LDAP integration 
*This document is intended to capture the steps of integrating LDAP users to manage CP4I.*


Refer to following link for detailed information: [Configuring an LDAP identity provider]()


## Configuring LDAP for Authentication

CP4I-LDAP Integration 

Login to Keycloak 

Click User Federation and enter your LDAP details.
Below screens are taken while configuring jumpcloud Ldap

<img width="1862" height="831" alt="Pasted Graphic 54" src="https://github.com/user-attachments/assets/abb84e97-3881-4060-a65c-e40b757f2115" />
￼
<img width="1862" height="831" alt="Pasted Graphic 55" src="https://github.com/user-attachments/assets/fc86c47e-5862-487e-830c-9be1e464cd23" />

<img width="1862" height="861" alt="Pasted Graphic 59" src="https://github.com/user-attachments/assets/631eb099-043b-4cdd-b8b6-468f03390009" />

<img width="1862" height="698" alt="Pasted Graphic 57" src="https://github.com/user-attachments/assets/b2adbfc0-0703-4c48-8b7e-3b82ac3ce9d6" />

<img width="1862" height="861" alt="Pasted Graphic 58" src="https://github.com/user-attachments/assets/5c9b9804-66bc-48c5-9cba-29e7fe6654fa" />


UPON clicking SAVE , the following screen is displayed

<img width="1872" height="434" alt="Pasted Graphic 60" src="https://github.com/user-attachments/assets/2fdbf214-fccd-4a53-98bd-fee8031c012f" />


Click Users to see the users imported from LDAP

<img width="1872" height="434" alt="Pasted Graphic 61" src="https://github.com/user-attachments/assets/5151bba1-7590-4491-a1aa-8074b8a128e0" />


When Trying to login to CP4I UI Console using the LDAP userId, if you get the Access Denied 403 error, it means that there is NO ROLE assigned to new User.

<img width="1872" height="861" alt="Pasted Graphic 62" src="https://github.com/user-attachments/assets/2cc29b9f-8e56-4579-a5a2-6b8f2558b660" />


Navigate to keyCloak Console UI

Goto Users—> Click on user (in this case we are using cp4i-admin)  —> Click on ‘Role Mapping’ tab

<img width="1903" height="853" alt="Pasted Graphic 64" src="https://github.com/user-attachments/assets/a5f575e8-a8bc-48ea-8728-235765e78813" />


Click on Assign Role and pick “integration-xxxx” - This will allow the new user full ADMIN access to CP4I UI similar to the original integration-admin user. 

<img width="1903" height="853" alt="Pasted Graphic 65" src="https://github.com/user-attachments/assets/cbae228c-520b-4957-b359-04f7e80ca7cd" />


Click Assign 

<img width="1903" height="443" alt="Pasted Graphic 66" src="https://github.com/user-attachments/assets/fd26a7a6-eaa6-4803-b930-ed199f958f04" />


Now navigate to CP4I url and login as the cp4i-admin user 

<img width="1903" height="890" alt="Pasted Graphic 63" src="https://github.com/user-attachments/assets/cc9af771-d5cc-4eb3-b4db-491904f9a46c" />






---
