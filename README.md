# *CAD Customer Recommendations Business Scenario Lab*

Note, this Lab can be used as an extension to the Personalisation lab which may be found here 
[Personalisation](https://github.com/shanepeckham/CADScenario_Personalisation). This will showcase a comprehensive and rich end to end scenario.

# What is it?

This repository will provision an environment that may be used as a Lab to build an end to end scenario that does the following:

*	Get the recommended products for a user based on product purchasing patterns. A sample dataset is provided but a custom one can be loaded too.
*	Perform automated transformation 
*	Prepare the output for a legacy website that is on-premise

# What does it showcase?

This solution brings together Infrastructure as a Service (IaaS), Platform as a Service (PaaS), Software as a Service (SaaS), Container as a Service (CaaS) and Serverless and Cognitive Services components on Microsoft Azure to build a realistic end to end scenario to generate recommendations for a customer based on other product purchase patterns using. This solution will show how an on-Premise XML File based legacy application can be quickly modernised to provide a microservice container based API for a front end such as a mobile application. Without needing to change any of the backend code, we will provide a modern microservice REST and JSON based API to front the legacy app.

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

### Now we need to install the On-Premise Data Gateway component within the Azure subscription

Select New in the Azure portal and enter *Data Gateway* and select the entry for 'On-premises Data Gateway', see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg11.png)
![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg12.png)

You will be presented with the configuration screen for the 'On-premises Data Gateway', ensure that you select the same *Resource Group* and *Location* that selected for the initial deploy and install, you should now see your Gateway appear in the *Installation Name* field, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dg13.png)

## 3. Now we will set up the Azure Container Registry

Navigate to Container Registry component provisioned within Azure, its name will be generated by default with the following format RecommACS[hash].

Click on the *Quick Start* blade, this will provide you with the relevant commands to upload a container image to your registry, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/quicksstartacs.png)

Now start a command/terminal window, create a directory called CADRecommendations and type the following:

``` 
docker pull shanepeckham\cadrecommendations
```
This will pull down the container image to your local machine that we will use for our API later on.

You should see a status similar to below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/dockerpullcad.png)

Now we will push the image up to the Azure Container Registry, enter the following (from the quickstart screen):

``` 
docker login recommacs[hash].azurecr.io

```

To get the username and password, navigate to the *Access Keys* blade, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/acskeys.png)

You will receive a 'Login Succeeded' message. Now type the following:
```
docker tag shanepeckham/cadrecommendations recommacs[hash].azurecr.io/cadrecommendations
docker push recommacs[hash].azurecr.io/cadrecommendations
```
Once this has completed, you will be able to see your container uploaded to the Container Registry within the portal, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/repos.png)

## 4. Train the Recommendations Model

Note, you can either train this model on your own Recommendations historic data or you can use the data in this lab, which is an extension of the data generated by the Bot in the [Personalisation](https://github.com/shanepeckham/CADScenario_Personalisation) lab.

More [info on the Recommendations API](https://westus.dev.cognitive.microsoft.com/docs/services/Recommendations.V4.0/) can be found here on all the methods: 

Navigate to Recommendations Cognitive Services component that was provisioned within Azure, its name will be generated by default with the following format RecommendationsCS[hash].

Click on the *Keys* blade and copy the value from one of the keys, this is an API Management authorisation key and will be required for every interaction, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/cskeys.png)

Now open Postman as we will use the REST API to train our model.

### Creating the model

Create a new request with the following parameters:

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models*
Request headers: 
        Content-Type: application/json
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
{
  "modelName": "CADRecommendations",
  "description": "CADRecommendations"
}
```
This will return the id for the model created, store this value, we will refer to it as modelId in the URI exmaples below.

### Upload the catalogue

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/catalog?catalogDisplayName=coffeecatalogue*
Request headers: 
        Content-Type: text
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
1, Latte, White
2, Americano, Black
3, Cappucino, White
4, Espresso, Black

```

We will use the contents of the [coffeecatalogue](https://raw.githubusercontent.com/shanepeckham/CADScenario_Recommendations/master/coffeecatalogue.txt) lab.
 file if you do not have specific catalogue to train.

### Upload the usage file

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/usage?usageDisplayName=coffeeusage*
Request headers: 
        Content-Type: text
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
2,2,2018-02-03 07:01:34
4,4,2017-10-14 13:57:27
4,1,2018-03-07 03:03:19
4,2,2017-01-10 21:05:20
1,1,2017-11-08 04:20:41
2,4,2017-03-30 08:07:27
1,3,2018-02-02 01:53:12
1,3,2017-04-10 06:08:44
1,2,2017-09-16 03:47:14
2,1,2016-12-13 13:12:45
3,3,2016-11-16 12:58:10
3,3,2017-03-24 17:21:38
3,3,2018-01-11 22:36:38
4,2,2017-01-19 20:54:27
4,1,2017-08-31 00:11:38
4,2,2016-12-05 12:25:08
2,1,2017-10-24 08:04:05
1,4,2016-12-13 00:11:22
2,2,2017-04-16 03:10:20
4,3,2017-07-13 03:47:20
3,3,2017-07-19 21:47:35
2,4,2016-11-08 19:22:37
2,3,2017-08-18 07:04:34
4,1,2018-02-19 15:35:53
2,4,2016-06-13 02:21:08
1,1,2017-09-01 20:00:08
1,1,2017-10-07 16:06:53
4,1,2017-04-11 08:05:58
4,4,2017-11-06 08:23:54
3,1,2017-10-26 17:57:24
3,3,2018-01-28 05:45:34
2,4,2017-07-07 12:38:42
3,2,2017-10-31 13:36:21
2,2,2016-06-16 08:39:35
2,2,2017-11-04 10:49:04
2,4,2017-11-11 19:06:58
4,3,2017-07-31 22:30:41
1,1,2017-01-13 16:07:23
1,1,2017-07-02 09:46:34
3,1,2017-06-15 16:20:18
4,2,2017-09-12 04:27:33
3,4,2016-07-01 19:17:01
3,4,2016-12-27 12:06:11
3,1,2017-01-11 04:44:14
4,3,2016-10-01 21:17:57
2,4,2017-03-14 00:54:48
4,1,2017-11-25 00:55:41
3,1,2016-08-28 11:29:28
2,4,2017-08-31 20:18:22
1,1,2016-06-07 18:46:15
2,2,2017-08-31 23:36:11
1,1,2017-05-28 00:08:37
2,1,2017-11-30 11:59:13
2,4,2018-02-10 09:01:37
2,1,2017-06-13 03:58:46
3,3,2018-03-26 15:55:39
1,1,2017-02-19 22:31:32
3,1,2016-10-09 17:03:38
3,4,2017-02-02 16:07:26
2,1,2017-03-15 02:10:54
2,3,2017-04-17 14:56:40
4,3,2016-07-10 02:05:46
2,1,2017-06-03 10:46:05
1,4,2017-02-01 05:10:36
1,1,2018-03-11 09:15:40
1,2,2017-01-20 11:55:16
3,1,2017-02-27 12:08:57
4,4,2016-10-23 07:24:00
3,1,2017-01-11 08:50:29
3,2,2016-10-29 18:28:02
1,3,2016-11-26 11:29:49
4,1,2017-01-02 19:26:17
2,3,2017-03-28 16:14:57
1,4,2017-09-12 00:19:02
1,3,2017-06-07 08:53:16
4,2,2017-02-08 03:18:41
4,4,2018-03-10 11:12:35
3,1,2017-05-11 18:35:32
4,2,2017-03-21 12:43:57
1,1,2016-10-14 05:23:42
4,2,2017-06-01 16:36:46
2,2,2017-05-06 05:28:17
1,1,2016-05-03 18:07:50
4,1,2018-04-17 12:12:05
2,2,2016-10-21 13:04:10
1,3,2016-06-18 21:01:02
1,1,2017-08-14 17:00:27
2,4,2017-11-08 18:20:58
1,4,2016-09-17 05:06:48
1,1,2016-12-31 10:01:16
3,1,2016-08-06 23:35:42
2,1,2017-12-01 15:03:38
1,3,2017-07-11 08:09:07
2,2,2016-09-17 01:23:41
1,2,2017-11-06 12:17:19
1,1,2016-06-29 23:24:06
1,4,2017-12-01 12:55:09
2,2,2018-01-05 17:15:14
4,2,2016-11-22 18:30:00
3,4,2016-12-02 09:57:41

```

We will use the contents of the [coffeecatalogue](https://raw.githubusercontent.com/shanepeckham/CADScenario_Recommendations/master/coffeeusage.txt) lab.
 file if you do not have specific catalogue to train.

 ### Create the Trigger/Build the model

HTTP Verb: POST
Request URL: *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/builds*
Request headers: 
        Content-Type: application/json
        Ocp-Apim-Subscription-Key: [RecommendationsCS.key]
Request body:
```
{

"description": "CAD: Simple recomendations build",

"buildType": "recommendation",

"buildParameters": {

    "recommendation": {

      "numberOfModelIterations": 20,

      "numberOfModelDimensions": 20,

      "itemCutOffLowerBound": 1,

      "itemCutOffUpperBound": 10,

      "userCutOffLowerBound": 0,

      "userCutOffUpperBound": 0,

      "enableModelingInsights": false,

      "useFeaturesInModel": false,

      "modelingFeatureList": "string",

      "allowColdItemPlacement": false,

      "enableFeatureCorrelation": true,

      "reasoningFeatureList": "string",

      "enableU2I": true

    }

  }

}

```
This will return a build Id which we will refer to as buildId.

## 5. Deploy the container to App Services

We will now deploy the container to Linux App Services. Navigate to your App Service App, it will have the name recommapp[hash, select the *Docker Container* menu option. We now need to add the provisioned container registry and uploaded container details. Enter the following values:

* Image Source: 'Private Registry'
* Image and optional Tag: ```recommendationsacs[hash].azurecr.io/cadrecommendations:latest``` This is your registry
* Server URL: ```https://recommendationsacs[hash].azurecr.io```
* Username: The username that is in the Container Registry Access Keys blade
* Password: The password that is in the Container Registry Access Keys blade

This will now deploy your container to App Services, it may take a few minutes so browsing to the site will return an HTTP 503 status during deployment. It is recommended that you navigate to the *Overview* section and Stop and then Start your API App as opposed to clicking the Restart button - a preview gotcha.

Now if you navigate to the site and add */docs* on to the end of the URL, you will see the swagger definition for the API where you can discover the methods and test them in the test harness, see below:

![alt text](https://github.com/shanepeckham/CADScenario_Recommendations/blob/master/images/swagger.png)
 
### Associate the Recommendations model URL with the API App

Our API app will check for two applications settings in order to be able to proxy the request to the Recommendations API, so we need to add these. Navigate to the *Application Settings* pane and add two entries in the 'App Settings' section, namely:

recommendationsAppURL  *https://westus.api.cognitive.microsoft.com/recommendations/v4.0/models/[modelId]/recommend/item?itemIds=*
recommendationsKey [RecommendationsCS.key]

Save the settings and navigate to the *Overview* section and Stop and then Start your API App as opposed to clicking the Restart button - a preview gotcha.

# The Lab component

We want to front our Container API behind API management and use API Management to convert the JSON return from the Recommendations API to XML. We need to do this using the API Management JSON-->XML policy. We also want to redirect the resultant XML response from the Container API to the On-Premise Windows Server File system so that it can be written for the legacy web application. To do this we need create a Logic App that will connect to the On-Premise file system using the On-Premise Data Gateway.

To summarise, the steps include:
* Place the Container API behind API Management
* Convert the response JSON to XML using an API Management Policy
* Invoke a Logic App from an API Management Policy and pass the XML to it
* Connect to the On-Premise Data Gateway with a Logic App and commit the XML file to the file System
* Return a response to API Management so that it can be passed to the Container API

# The Lab solution

## SPOILER ALERT - this is the full solution, so only look here if you get stuck!

### The data model



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




