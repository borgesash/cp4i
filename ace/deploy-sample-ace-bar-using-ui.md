# ACE BAR file deploying using ACE Dashboard UI


## Introducing the App Connect Dashboard

The App Connect Dashboard is a user interface that provides a simplified way to deploy BAR files to IBM App Connect containers, and to administer the deployed integration servers or integration runtimes.

## Creating an App Connect Dashboard instance
The App Connect Dashboard is provided as part of an IBM App Connect Operator installation, so it is assumed that you have installed this Operator as described in Access to an installation of the IBM App Connect Operator. The Operator doesn't automatically create or install an instance of the App Connect Dashboard (or any of the other custom resources), so you’ll need to now create the Dashboard instance that you'll use for the deployment.

## Accessing the App Connect Dashboard
You can access the Dashboard instance from the Platform UI in your IBM Cloud Pak for Integration environment, or from the Red Hat OpenShift web console.

### Accessing the Dashboard from the Platform UI
When you log in to the Platform UI, you can directly open the App Connect Dashboard instance as shown below

<img width="1860" height="670" alt="Pasted Graphic 75" src="https://github.com/user-attachments/assets/66ca5d12-519c-4f88-9d03-7835304008ce" />

### Accessing the Dashboard from the Red Hat OpenShift web console
From the namespace (or project) where you created the App Connect Dashboard instance (under Operators > Installed Operators > IBM App Connect), click the Dashboard tab and then click the Dashboard name.
On the Details tab of the ““Dashboard details”” view, the Admin UI field provides the URL for accessing the Dashboard instance. You can right-click this URL and then use the Open link in new tab option to open the Dashboard on a separate tab.

<img width="966" height="535" alt="Dashboard overview" src="https://github.com/user-attachments/assets/c58a4fd7-918a-4d34-a7c1-098b944d2298" />

## Exploring the App Connect Dashboard
The Dashboard home page displays the following tiles:
- A Deploy integrations tile that you can use to create an integration runtime
- A Learn more tile that you can use to view the IBM App Connect in containers documentation
- A Runtimes tile that indicates how many integration runtimes are running, and which you can use to view the deployed runtimes
- An Integrations tile that indicates how many integration applications are currently deployed for the integration runtimes, and which you can use to view these integrationsThe Runtimes and Integrations tiles are shown only if integration runtimes have been deployed.


### Obtaining the BAR file that you want to deploy
You are now going to deploy this [HttpEchoApp BAR](https://github.com/amarIBM/hello-world/blob/master/HttpEchoApp.bar) file by using the App Connect Dashboard. You will upload the BAR file to the Dashboard's internal content server for storage instead of referencing the file from an external location.
The file is provided as an attachment for your use: [HttpEchoApp.zip](https://www.ibm.com/docs/en/SSTTDS_13.0/containers_cd/com.ibm.ace.icp.doc/HttpEchoApp.zip). Download this attachment and then extract its contents to a directory on your local computer. This ZIP file contains the HttpEchoApp.bar file, which you will use in the next procedure.

### Deploying the BAR file to an integration runtime by using the App Connect Dashboard

From the home page of the App Connect Dashboard, start the integration runtime deployment by clicking the Deploy integrations tile.

<img width="1860" height="806" alt="Pasted Graphic 59" src="https://github.com/user-attachments/assets/b7768ee7-92b2-480d-adc8-d66f71413ab6" />


From the Size view, click the Quick start integration tile because you are deploying a BAR file that contains a simple integration.
(The tiles in this view present different sizing options, and you can review the sizings on each tile to help you choose a suitable option for the integration that you want to run. These integrations can typically originate from either App Connect Designer or the IBM App Connect Enterprise Toolkit.)

<img width="1860" height="806" alt="Pasted Graphic 60" src="https://github.com/user-attachments/assets/4262d9b6-3e1d-4864-95e0-5a945dc2f0fa" />


Click Next to proceed.
From the Integrations view, upload the HttpEchoApp.bar file (which you downloaded earlier) to the Dashboard's content server. You can drag this file into the boxed area, or click to select and upload the file from its extracted location.

<img width="1860" height="806" alt="Pasted Graphic 61" src="https://github.com/user-attachments/assets/2f845ed0-2ec4-4958-99c4-13c3cc7975de" />


Click Next to proceed to the Configuration view, which can be used to define dynamic configurations that you would like to apply to integration runtimes that you are deploying.
For more information about the configuration types that you can create from the Dashboard, see [Learn more about the configuration types](https://www.ibm.com/docs/en/SSTTDS_13.0/containers_cd/com.ibm.ace.icp.doc/config_configuration.html#config_configuration__config_learnmore).


<img width="1860" height="510" alt="Pasted Graphic 62" src="https://github.com/user-attachments/assets/74f19431-db00-4981-80ba-9278f839756e" />

Click Next to proceed to the Properties view, where you can provide metadata for your integration runtime and install the BAR file resources. Specify the following values and retain the default values in any remaining fields. (Notice that you do not need to specify a namespace because the integration runtime is automatically deployed in the same namespace as the Dashboard.)

<img width="1860" height="797" alt="Pasted Graphic 63" src="https://github.com/user-attachments/assets/2b3400c1-4254-4bed-837e-3295c6e79d99" />

<img width="1860" height="797" alt="Pasted Graphic 64" src="https://github.com/user-attachments/assets/f44c5407-65f1-4458-8c64-ccd9c9ad2101" />


Click Create to create the integration runtime.
When the deployment completes, the integration runtime is displayed as a tile on the Runtimes page. The status, which is initially shown as Pending, will change to Ready when the deployment completes

<img width="1860" height="797" alt="Pasted Graphic 66" src="https://github.com/user-attachments/assets/309726fb-2bbb-40df-aed8-6ff538c10883" />

<img width="1860" height="797" alt="Pasted Graphic 67" src="https://github.com/user-attachments/assets/bcd950a4-29ed-4fd3-a66d-ba22bc06deed" />


Drill down into the integration runtime to view the deployed applications and the message flows under each application. To do so, click the integration runtime tile on the Runtimes page to view the tile and tabs for the defined application. Then, click the application tile to view the tile and tabs for the message flow.
(From the Runtimes page, you can also view all deployed integrations by clicking the Integrations tab to open the Integrations page.)

<img width="1860" height="797" alt="Pasted Graphic 68" src="https://github.com/user-attachments/assets/4c2b2eec-8e86-41e8-9e35-3aee519b8052" />

<img width="1860" height="797" alt="Pasted Graphic 69" src="https://github.com/user-attachments/assets/29ee2eaf-4851-47cb-901d-97ec2c2e84b9" />

<img width="1860" height="828" alt="Pasted Graphic 70" src="https://github.com/user-attachments/assets/8c599fe7-8449-4309-bbf8-33da2aebb770" />


### Testing the message flow
Click on the Endpoints to determine the URL to invoke the service.

<img width="1860" height="458" alt="Pasted Graphic 72" src="https://github.com/user-attachments/assets/c7972c19-3bdd-48e1-80d1-5f705013dd01" />


Invoke the service URL from a browser address bar.

The response should look like this, which indicates that you successfully invoked your deployed flow.

<img width="1860" height="288" alt="Pasted Graphic 73" src="https://github.com/user-attachments/assets/c525b2e5-a12b-479e-9bce-cfe6ff1a5896" />


By completing this deployment from the App Connect Dashboard, you can see that the Dashboard enables you to perform the same actions on the underlying Kubernetes platform as you completed by using the command line. You can switch between the CLI and the Dashboard user interface as preferred, and should observe that the results of any actions in one interface are reflected in the other.

## References
https://www.ibm.com/docs/en/app-connect/13.0.x?topic=dtiir-scenario-2-deploying-toolkit-integration-from-app-connect-dashboard-1


