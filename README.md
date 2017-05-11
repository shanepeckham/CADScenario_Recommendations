# *CAD Customer Recommendations Business Scenario Lab*

Note, this Lab can be used as an extension to the Personalisation lab which may be found here 
[Personalisation](https://github.com/shanepeckham/CADScenario_Personalisation). This will showcase a comprehensive and rich end to end scenario.

# What is it?

This repository will provision an environment that may be used as a Lab to build an end to end scenario that does the following:

*	Get the recommended products for a user based on product purchasing patterns. A sample dataset is provided but a custom one can be loaded too.
*	Perform automated transformation 
*	Prepare the output for a legacy website that is on-premise

# What does it showcase?

This solution brings together Infrastructure as a Service (IaaS), Platform as a Service (PaaS), Software as a Service (SaaS), Container as a Service (CaaS) and Serverless and Cognitive Services components on Microsoft Azure to build a realistic end to end scenario to generate recommendations for a customer based on other product purchase patterns using. This solution will show how an on-Premise XML File based legacy application can be quickly modernised to provide a microservice container based API for a front end such as a mobile application.

# The end to end scenario

This solution will determine the list of recommended products for a particular customer and return the response in JSON. An asynchronous process will convert the message to XML and write it back to an On-Premise legacy XML File system.

# The solution aims to show the following:

*	How legacy lift and shift applications on IaaS can be incorporated into modern solutions to quickly derive value from higher value services in the cloud.
*	How existing investments can be modernized without having to rebuild everything to drive customer value
*	The ease with which On-premise, public and private components may be brought together to build workloads that bring business value
*	The meshing of IaaS, PaaS, SaaS, Serverless and AI with tools that are accessible to non-developers
*	OSS workloads running on Azure

# Technology used

The following technology components are used in this solution:

*	Swagger enabled Node.js APIs running in a container on the Azure Linux App Services (PaaS) (CaaS)
*   The Azure Container Registry to privately and securely store approved container images
*	A legacy Windows machine running on-Premise or on a VM that serves as an XML File system for a conceptual legacy website.
*	Azure networking to isolate legacy workloads (IaaS)
*	API Management to govern APIs and to perform automated JSON<-->XML conversion and IP Rate limiting
*	Azure Logic apps to provide serverless integration that is accessible to non-developers (Serveless)
*   The Logic apps on-Premise Data Gateway to connect to an on-Premise machine or a VM in an isolated VNET
*	Azure Resource Manager templates to automate the provisioning and inflation of a full environment

# Solution flow

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/Topology.png "Solution Flow")

# The Lab component

This solution will install most of the components required to build the end to end Recommendations scenario. The Lab attendees need to configure a few components, install the on-Premise Data Gateway, prepare an API Management Policy and wire everything together in a Logic App. 

# Preparing for the solution

For this Lab you will require (note, these items can be all be installed on the VM that will be provisioned during the lab):

* Install Postman, get it here - https://www.getpostman.com
* Install Docker, get it here - https://docs.docker.com/engine/installation/
* Install the Logic App On-Premise Data Gateway, get it here - https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install

# How to install the solution

## 1. Provisioning the components: Select Deploy to Azure to deploy to your Azure instance that you are currently logged in to.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fshanepeckham%2FCADScenario_Recommendations%2Fmaster%2Fazuredeployrecommendations.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fshanepeckham%2FCADScenario_Recommendations%2Fmaster%2Fazuredeployrecommendations.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

Select or create a Resource Group to deploy to and the only parameter you need to change is the Deployment Name - give it any name of 12 characters or less as it will be used to generate a hash to ensure your site names are unique, see the image below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/DeploymentName.png)

This will take roughly 20-30 minutes as this will provision:

* One VNET
* A Windows VM and place in inside the VNET isolated with NSGs
* An API Management instance (Developer Tier)
* An Azure Container Service Registry
* A Linux App Service API app 
* Storage accounts to house the VM VHD, the App Service API logging and the Azure Container Registry
* A Cognitive Services account configured for the Recommendations API

## 2. Installing the on-Premise Data Gateway for Logic Apps

Navigate to your VM, the default name will be LegacyXML[hash] and navigate to the Overview blade and copy the value in the field Public IP Address/DNS label, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/legacyip.png)

Connect to the machine via Remote Desktop with the following credentials:
User: MiniCADAdmin
Password: MiniCADAdmin123

***Take note of the value in the domain as you may need to use this later

Navigate to https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install and download the Data Gateway.

Click next on the first step of the install, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg1.png)

Accept the default install location and accept the terms and conditions, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg2.png)

You should now get to a screen that states that the install was successful, now you need to enter an email address to identify the gateway account, use the email address associated with your Azure subscription and click 'Sign in', see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg3.png)

Complete the logon process to Azure.

Now you need to enter the Gatewat Server Name and a recovery key, see below:

Enter *LegacyXML* as the Gateway Name and select a recovery key that you won't forget and copy it somewhere just in case. Ensure that the region highlighted below is the same as the region that you selected to deploy the components to from Step 1 above, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg4.png)

The image below shows the region change option:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg5.png)

You should get a success screen as below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg6.png)

### Now we need to create a share that will hold the XML Files for the legacy application

Open up Windows Explorer and create a new folder on the C:\ drive called *ShoppingXML*, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg7.png)

Now right click the ShoppingXML directory and select Properties --> Sharing, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg8.png)

Select 'Share' and highlight the *MiniCADAdmin* user to share with and click the 'Share' button on the bottom right of the popup, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg9.png)

You should now see that the folder has been shared, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg10.png)

### Change the email addresses to in the contact API to your email

* Navigate back to your API App in the Azure portal - e.g. http://cadapimastersite[hash].azurewebsites.net
* Select the Advancted Tools blade and then click Go. This will open the Kudu console where we can be naughty and go and edit the source files 
* In the top menu select Debug Console --> CMD
* In the top tree folder structure click Site --> WWWRoot --> lib -- and click the pencil next to the contacts.json file. 
* Change the email for the contact you want to change
* Click Save
* Restart your API App. This can be done from the Restart button on the overview blade

## 3. Now we will import the Contact List API into the API Management solution

Navigate to API Management component provisioned within Azure, its name will be generated by default with the following format cadapim[hash].

Click on the overview blade and select 'Publisher Portal' (note we will use the old portal while the new portal is still in preview), see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/apimpublisherportal.png)

This will navigate you to the Publisher portal, select Import API, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/importAPI.png)

Now enter the following values:

* Select "From Url"
* Paste your API URL from step 2 and add ``` /swagger ``` on the end e.g. http://cadapimastersite[hash].azurewebsites.net/swagger
* Select "Swagger" as the specification format
* Select New API
* Type "Contacts" into the Web API URL Suffix field
* Click Products and select "Unlimited"
* Click Save

See below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/importapidetails.png)

You will now have imported an API that will now be accessible from the API Gateway. You can test it by clicking on Developer Portal, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/developerportal.png)

Click APIs --> Contact List API --> Try It

This will take you to the test harness of API Management. Click the eyeball and copy the value in the field Ocp-Apim-Subscription-Key, this is your APIMKey which we will use extensively, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/apimkey.png)

You can test the API now and it should return values with a status of 200 Ok.

We have now set up the first API in our process.

## 4. Install the legacy Ticket API on the VM

We could deploy this script as a custom script extension on the VM but that will complicate troubleshooting in a lab scenario so we will manually connect to the machine and run the build script, it is a single install script that will set up everything required.

Navigate to your VM, the default name will be CADLegacyAPI[hash] and navigate to the Overview blade and copy the value in the field Public IP Address/DNS label, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/vmip.png)

We will now ssh onto the machine using Bash for Windows on Windows 10, or putty or just plain old terminal on a mac or Linux.

* Type ssh MiniCADAdmin@[pasted ip address - without value '/none' on the end) e.g. ssh MiniCADAdmin@12.34.56.78 and press enter - see below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/ssh2.jpg)

* Select yes to the message "Are you sure you want to continue connecting"
* Type in password MiniCADAdmin123 - note this is hardcoded in the deploy
* Paste the following in the command line: ``` git clone https://github.com/shanepeckham/CADHackathon_Loyalty.git ```
* Now type ``` cd CADHackathon_Loyalty ```
* Now type ``` bash installVM.sh ```
* Upon completion you will see a screen similar to that below, with the final status 'Starting Legacy API'

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/startinglegacyapi.png)

Your Legacy Ticket API should now be listening on port 8000 but this will not be accessible from the outside world. Note, if you restart your VM or want to restart the legacy api, simply navigate to the /LegacyAPI/CADContacts folder and run node server.js. 
e.g.

```cd LegacyAPI 
cd CADContacts
node server.js
```
 
## 5. Now we will import the Case Contact List API (Legacy Ticket API) into API Management.

Navigate to API Management component provisioned within Azure, its name will be generated by default with the following format cadapim[hash].

Click on the overview blade and select 'Publisher Portal' (note we will use the old portal while the new portal is still in preview), see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/apimpublisherportal.png)

This will navigate you to the Publisher portal, select Import API, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/importAPI.png)

Now enter the following values:

* Select "From Url"
* Paste http://10.1.1.4:8000/swagger into this field. Note this is your internal VNET address for your VM
* Select "Swagger" as the specification format
* Select New API
* Type "LegacyAPI" into the Web API URL Suffix field
* Click Products and select "Unlimited"
* Click Save

See below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/legacyapiimport.png)

You will now have imported an API that will now be accessible from the API Gateway. You can test it by clicking on Developer Portal, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/devportallegacy.png)

Click APIs --> Contact Case List API --> Try It

This will take you to the test harness of API Management. Click the eyeball and copy the value in the field Ocp-Apim-Subscription-Key, this should be the same as your previous API product.

You can test the API now and it should return values with a status of 200 Ok.

We have now set up the legacy Ticket API in our process.

## 6. Import Function Storage hooks

Click on the Deploy button below.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fshanepeckham%2FCADHackathon_Loyalty%2Fmaster%2Fazuredeployfunctionsettings.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

NB - Make sure you use the same Deployment Name as you did in Step 1.

Update: Azure functions has recently changed and the new release is not deploying the code correctly. Navigate to Platform Features --> Deployment Options, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/newfuncdeploy1.png)

In the pane on the right click the 'Sync' button. This will redploy the code and will take around 2-3 minutes, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/newfuncdeploy2.png)

Once complete, click the Refresh icon as highlighted below and you should see the following:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/newfuncdeploy3.png)

You can test the function in the test harness within the function app itself, add this to the Request body:
```
{ "name": "hello" }
```
You should see the output look something like this:
```
{"couponUrl":"https://cadfuncstorrvhyzok7zv4gw.blob.core.windows.net/coupons/%5Bobject%20Object%5D.jpg?st=2017-03-14T20%3A27%3A59Z&se=2017-03-14T21%3A27%3A59Z&sp=r&sv=2015-12-11&sr=b&sig=6UUHFHY08JihUU8vT%2Fus%2Fot9Pl%2BZud6jaakMNTuCFZc%3D"} Status 200 Ok
```
Note, if you get an error upon first invocation, run it again and it should work. You are now ready to build the logic app.

# The Logic App solution

## SPOILER ALERT - this is the full solution, so only look here if you get stuck!

We want to get the customer's details, find their last associated case and then check the feedback against it.

### The data model

See the diagram below for the simplistic data model to help you query the right data.

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/DemoDataModel.jpg)

Create a HTTP Request Step, click save - you will receive an endpoint upon save. 

Select the Request step, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/Request.png)

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/httprequest.png)

Once you save the Logic app you will get and endpoint, you can now invoke your logic app with Postman - add the URL and select POST. Ensure you have set the Header "Content-Type" with value "application/json". Select body, select "raw" and enter the follow value for your body content:
```
{
  "APIMKey": "[Your APIM Key goes here]",
  "id": 1
}
```
![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/postmanheaders.png)
![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/postmanbody.png)

Now add a step to include an API Management API - select your API "Contact List API". You will want to select the method GET for contacts/{id}

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/contactsstep1.png)

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/contactsstep2.png)

You will need to navigate to the code view to be able to select the json fields that will be posted as part of the body. Note, you can select the values you are posting in the body using javascript object notation. 

Your code view should look like this:
```
            "QueryContactsById": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/[ResourceGroup]/providers/Microsoft.ApiManagement/service/cadapimxdb3o43h6p7bq/apis/58cd4c45dc78ac0f84da1287"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(triggerBody()['id'])}"
                        },
                        "template": "/Contacts/contacts/{id}"
                    },
                    "subscriptionKey": "@{triggerBody()['APIMKey']}"
                },
                "runAfter": {},
                "type": "ApiManagement"
            }
```
Navigating back to the designer should show your values resolved like below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/querycontactsbyid.png)

Now you need to query the Legacy Ticket API (Contacts Case List API) which is inside the isolated network to get the last case for that customer and retrieve the customer's feedback on the case. From the demo model hint above you want to use the caseNum field from the QueryContactsById to query the Legacy Ticket API field primary key field CaseId.

Add an API Management API step, select the Contact Case List API and once again query by Id, which in this CaseId which maps to the caseNum output from the previous step and add the API Management subscription key.

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/querycasesbyid.png)

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/querycasesselectidmethod.png)

Ensure you select the correct outputs from the previous step(s) as inputs to this step, see below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/querycasebycasenum.png)

Now if you applied the logic from the previous step to select the correct outut field values (which would be a sensible approach) you will probably get the following error:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/querycaseserror.png)

This is due to us outputting an array but not specifying which record/index we want to use. Change the following line from
```
"id": "@{encodeURIComponent(body('QueryContactsById')?['caseNum'])}"
```
to use the first value of the array, namely the 0 index:

```
  "id": "@{encodeURIComponent(body('QueryContactsById')[0]['caseNum'])}"
```

Here is what your code view should look like for this step:
```
            "QueryCasesById": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/[ResourceGroup]/providers/Microsoft.ApiManagement/service/cadapimxdb3o43h6p7bq/apis/58cd5516dc78ac0f84da1289"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(body('QueryContactsById')[0]['caseNum'])}"
                        },
                        "template": "/LegacyAPI/contacts/{id}"
                    },
                    "subscriptionKey": "@{triggerBody()['APIMKey']}"
                },
                "runAfter": {
                    "QueryContactsById": [
                        "Succeeded"
                    ]
                },
                "type": "ApiManagement"
            },
```

Now we want to add our Cognitive Services 'Detect Sentiment' (note you will need to have your key ready that you got when you signed up for the Text Analytics preview as part of the pre-requisites) step  so that we can analyse the sentiment of the Ticket Feedback:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/searchdetect.png)

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/selectsentiment.png)

We now need to make sure we send the correct output from the QueryCasesById step to the Detect Sentiment step, use the javascript dot notation approach again with an index value. Your steps should look like this:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/lastfeedback.png)

Your code view should look like this:
```
            "Detect_Sentiment": {
                "inputs": {
                    "body": {
                        "text": "@{body('QueryCasesById')?['last_feedback']}"
                    },
                    "host": {
                        "api": {
                            "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/cognitiveservicestextanalytics"
                        },
                        "connection": {
                            "name": "@parameters('$connections')['cognitiveservicestextanalytics']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/sentiment"
                },
                "runAfter": {
                    "QueryCasesById": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
```

Now we want to add a condition to check the sentiment, if the probability outcome is less than 0.5, then it negative sentiment and therefore qualifies for our discount coupon.

Your condition should look like this:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/Condition.jpg)

With the following in code view:
```
"expression": "@less(body('Detect_Sentiment')?['score'], 0.5)",
                        "runAfter": {
                            "Detect_Sentiment": [
                                "Succeeded"
                            ]
                        },
                        "type": "If"

```

Now you can call the GenerateCoupon function if the condition is met, pass in the name of the user that you want to generate the digital coupon for:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/ifcondition0.png)

With the following in the code view:
```
"GenerateCoupon": {
                        "inputs": {
                            "body": {
                                "name": "@body('QueryContactsById')[0]['name']"
                            },
                            "function": {
                                "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.Web/sites/CADFuncxdb3o43h6p7bq/functions/GenerateCoupon"
                            }
                        },
                        "runAfter": {},
                        "type": "Function"
                    }
                },
```
Now we want to send an email to every receipient to inform them that they can download a digital coupon which we have generated for them.

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/selectgmail.png)

Your email step should look like this:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/sendemail.png)

With the following code view:
```
                    "Send_email": {
                        "inputs": {
                            "body": {
                                "Body": "Please accept this coupon: @{body('GenerateCoupon')}",
                                "Subject": "We heard that you were not happy",
                                "To": "@{body('QueryContactsById')[0]['email']}"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/gmail"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['gmail']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/Mail"
                        },
                        "runAfter": {
                            "GenerateCoupon": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    }
```

The full code solution view looks like this:
```
{
    "$connections": {
        "value": {
            "cognitiveservicestextanalytics": {
                "connectionId": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.Web/connections/cognitiveservicestextanalytics",
                "connectionName": "cognitiveservicestextanalytics",
                "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/providers/Microsoft.Web/locations/northeurope/managedApis/cognitiveservicestextanalytics"
            },
            "gmail": {
                "connectionId": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.Web/connections/gmail",
                "connectionName": "gmail",
                "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/providers/Microsoft.Web/locations/northeurope/managedApis/gmail"
            }
        }
    },
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "Condition": {
                "actions": {
                    "GenerateCoupon": {
                        "inputs": {
                            "body": {
                                "name": "@body('QueryContactsById')[0]['name']"
                            },
                            "function": {
                                "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.Web/sites/CADFuncxdb3o43h6p7bq/functions/GenerateCoupon"
                            }
                        },
                        "runAfter": {},
                        "type": "Function"
                    },
                    "Send_email": {
                        "inputs": {
                            "body": {
                                "Body": "Please accept this coupon: @{body('GenerateCoupon')}",
                                "Subject": "We heard that you were not happy",
                                "To": "@{body('QueryContactsById')[0]['email']}"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/gmail"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['gmail']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/Mail"
                        },
                        "runAfter": {
                            "GenerateCoupon": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    }
                },
                "expression": "@less(body('Detect_Sentiment')?['score'], 0.5)",
                "runAfter": {
                    "Detect_Sentiment": [
                        "Succeeded"
                    ]
                },
                "type": "If"
            },
            "Detect_Sentiment": {
                "inputs": {
                    "body": {
                        "text": "@{body('QueryCasesById')?['last_feedback']}"
                    },
                    "host": {
                        "api": {
                            "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/cognitiveservicestextanalytics"
                        },
                        "connection": {
                            "name": "@parameters('$connections')['cognitiveservicestextanalytics']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/sentiment"
                },
                "runAfter": {
                    "QueryCasesById": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
            "QueryCasesById": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.ApiManagement/service/cadapimxdb3o43h6p7bq/apis/58cd5516dc78ac0f84da1289"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(body('QueryContactsById')[0]['caseNum'])}"
                        },
                        "template": "/LegacyAPI/contacts/{id}"
                    },
                    "subscriptionKey": "@{triggerBody()['APIMKey']}"
                },
                "runAfter": {
                    "QueryContactsById": [
                        "Succeeded"
                    ]
                },
                "type": "ApiManagement"
            },
            "QueryContactsById": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/1b987fd6-b38e-40a1-bca8-4f67e6272c12/resourceGroups/NewLoyaltyPlan/providers/Microsoft.ApiManagement/service/cadapimxdb3o43h6p7bq/apis/58cd4c45dc78ac0f84da1287"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(triggerBody()['id'])}"
                        },
                        "template": "/Contacts/contacts/{id}"
                    },
                    "subscriptionKey": "@{triggerBody()['APIMKey']}"
                },
                "runAfter": {},
                "type": "ApiManagement"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "manual": {
                "inputs": {
                    "schema": {}
                },
                "kind": "Http",
                "type": "Request"
            }
        }
    }
}

```
# Troubleshooting

If you get and 'Internal Server Error' 500 on the Contact Case List (Legacy Ticket API) this could be because the node API has stopped. ssh into the VM and navigate to the /LegacyAPI/CADContacts folder and run 'node server.js'. 
e.g.

```cd LegacyAPI 
cd CADContacts
node server.js
```

If you get a 404 - Resource not found within the Logic app invoking the Legacy API, ensure that you have selected *HTTPS* not *HTTP* as the Web API URL Scheme. Check your settings are the same as in this image below:

![alt text](https://github.com/shanepeckham/CADLab_Loyalty/blob/master/Images/troubleshoot1.png)


# Securing this environment

The security lab for this environment may be found here [Securing the Loyalty environment](https://github.com/shanepeckham/CADLab_Loyalty_Security) 




