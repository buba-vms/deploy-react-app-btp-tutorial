# Deploying react app to sap btp

Repository created to demonstrate how you can deploy a react app using xsuaa authentication in sap btp.

**Disclaimer** - I would like to share this tutorial that I created after having a hard time deploying a react app to sap btp using xsuaa authentication. I want to make it clear that I am still a beginner in the field, therefore, this tutorial may not be the most efficient or complete solution. However, I'm sharing my learning in the hope of helping other people that may be facing the same difficulties that I did. Feel free to share your tips and feedback.

# Introduction

Welcome to this tutorial where I'll try to explain the best that I can how to deploy a React app to SAP BTP using the XSUAA Service, the approuter configuration are mainly from this [video](https://www.youtube.com/watch?v=Jbq6zaKEUyY) from SAP Developers Youtube Channel, where Nico Schoenteich explains very well, how to config the approuter, and I highly recommend you to watch it, but the tutorial that he did is for UI5 application, so I had to make some adjustments to make it work for a react app.
# Prerequisites

* [Basic Understanding of SAP BTP](https://help.sap.com/docs/btp/sap-business-technology-platform/btp-basic-platform-concepts?locale=en-US)
* [SAP BTP Trial Account](https://developers.sap.com/tutorials/hcp-create-trial-account.html)
* [Node.js](https://nodejs.org/en)
* [Cloud Foundry CLI](https://developers.sap.com/tutorials/cp-cf-download-cli.html)

# Setup

* Create a folder for the full project e.g. `deploy-react-app-btp-tutorial`
* Open it in your favorite IDE
* Open your terminal and make the following command
```
cf login
```

# Services

* inside `deploy-react-app-btp-tutorial` folder, create a `service-instances` folder you'll need it for creating the xsuaa service

### HTML5 Applications
* To display or run HTML5 applications, please subscribe to at least one of the following services, I subscribed to the SAP Build Work Zone, standard edition.

![image](https://user-images.githubusercontent.com/101530871/233108262-7754139d-83ad-41cb-8801-6e2b69fd7b1b.png)

### ABAP Envoirment
* On btp cockpit, create an instance of ABAP Environment
![image](https://user-images.githubusercontent.com/101530871/233134090-57e54bec-b91a-4db1-ad65-43b08fb7c463.png)

![image](https://user-images.githubusercontent.com/101530871/233135584-b60f8bde-5cbc-4c8f-ba49-3c6109dfbfbb.png)

![image](https://user-images.githubusercontent.com/101530871/233135823-7666b2fb-c836-45eb-b8a2-b2ca5a93c093.png)

![image](https://user-images.githubusercontent.com/101530871/233135923-bab518ff-99e3-4079-a7b4-a003edd82fa7.png)

![image](https://user-images.githubusercontent.com/101530871/233136027-a962f774-57cb-4288-b728-3fa17545b67e.png)

![image](https://user-images.githubusercontent.com/101530871/233136186-17759506-0795-40f4-b3a1-0045e1715653.png)

* now we'll have to create a service-key 

![image](https://user-images.githubusercontent.com/101530871/233137962-f7228bf6-9452-43d2-9f13-4cca5601e70d.png)

![image](https://user-images.githubusercontent.com/101530871/233138191-4a927858-d7dd-4069-b0ad-7f019084de6f.png)

![image](https://user-images.githubusercontent.com/101530871/233138366-4796e517-f5a2-428b-854b-4bfba1ac6baf.png)


### XSUAA

* inside `service-instances` create an `xs-security.json` file with the following content
```json
{
  "xsappname": "tutorial-xsuaa",
  "tenant-mode": "dedicated",
  "oauth2-configuration": {
    "redirect-uris": [
      "https://tutorial-approuter-*cfapps.us10-001.hana.ondemand.com/**"
    ]
  }
}
```
* Open the terminal on the `service-instances` folder and insert the following command
```
cf create-service xsuaa application tutorial-xsuaa -c xs-security.json
```
* you should receive the following response

![image](https://user-images.githubusercontent.com/101530871/233129616-184100db-26b3-464a-9d16-6f7b3a5111cb.png)

### Destination
* inside `service-instances` create a `dest-config.json` file with the following content
```json
{
  "init_data": {
    "instance": {
      "existing_destinations_policy": "update",
      "destinations": [
        {
          "Type": "HTTP",
          "clientId": "insert your client id here",
          "clientSecret": "insert your client secret here",
          "Authentication": "OAuth2UserTokenExchange",
          "Name": "abap-destination",
          "tokenServiceURL": "insert your tokenServiceURL here",
          "ProxyType": "Internet",
          "URL": "Insert your URL here",
          "tokenServiceURLType": "Dedicated"
        }
      ]
    }
  }
}
```
* to find the information above you have to click to view the credentials of abap environment key that we created
![image](https://user-images.githubusercontent.com/101530871/233141067-db8c7c1e-d0c1-4e51-b310-b57e676f2d0b.png)

* Open the terminal on the `service-instances` folder and insert the following command
```
cf create-service destination lite tutorial-destination -c dest-config.json
```
* you should receive the following response

![image](https://user-images.githubusercontent.com/101530871/233142111-ded50c92-968f-4b73-864b-878b8977176d.png)

### HTML5 Runtime

* Open the terminal and insert the following command
```
cf create-service html5-apps-repo app-runtime tutorial-html5-runtime
```
* you should receive the following response

![image](https://user-images.githubusercontent.com/101530871/233144104-9a554596-d093-4336-9d91-ce32bbfdce88.png)

### HTML5 Host

* Open the terminal and insert the following command
```
cf create-service html5-apps-repo app-host tutorial-html5-host
```
* you should receive the following response

![image](https://user-images.githubusercontent.com/101530871/233156516-f6601f4d-a849-454f-85e3-ef8fe8a78182.png)


# Approuter
* inside `deploy-react-app-btp-tutorial` folder, create an `approuter` folder
* inside `approuter` folder create a `package.json` file with the following content
```json
{
  "name": "tutorial-approuter",
  "scripts": {
    "start": "node node_modules/@sap/approuter/approuter.js"
  },
  "dependencies": {
    "@sap/approuter": "^9"
  }
}
```
* every approuter needs a file called `xs-app.json` that describes its routing behavior, so create it inside the `approuter` folder with the following content
cf create-service html5-apps-repo app-host tutorial-html5-host
```json
{
  "welcomeFile": "tutorial/index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^/abap(.*)",
      "target": "$1",
      "destination": "abap-destination",
      "authenticationType": "xsuaa"
    }
  ]
}
```

* finally you'll need a `manifest.yaml` file to deploy the approuter to btp using Cloud foundry, so create it inside `approuter` with the following content
```yaml
applications:
  # Application Router
  - name: tutorial-approuter
    buildpacks:
      - nodejs_buildpack
    memory: 256MB
    disk_quota: 256MB
    random-route: true
    path: .
    services:
      - tutorial-xusaa
      - tutorial-destination
      - tutorial-html5-runtime

```
* Open the terminal inside the `approuter` folder and type the command bellow
```
cf push approuter
```
if you did every configuration of the services, you should receive the following response
![image](https://user-images.githubusercontent.com/101530871/233145690-1e7a8664-d3ff-49d4-8af3-65865acd9e01.png)

if you go to your applications on your dev space you should see your approuter running 
![image](https://user-images.githubusercontent.com/101530871/233147308-62387f2c-05b5-4ab6-a1b4-dd0b48b2af3c.png)

if you go to the URL of your approuter and make the login you should see this, not found is expected because we still didn't deploy our react app
![image](https://user-images.githubusercontent.com/101530871/233150016-b90429ef-9ac4-44d9-9a3c-0cc403ecdbb4.png)


# Setup of react app
* Open the terminal inside the `deploy-react-app-btp-tutorial` folder and type the command bellow
```
npx create-react-app app
```
* inside the `public` folder of your react app create a `manifest.json` with the following content
```json
{
  "version": "1.42.0",
  "sap.app": {
    "id": "tutorial",
    "type": "application",
    "i18n": "i18n/i18n.properties",
    "applicationVersion": {
      "version": "1.0.0"
    },
    "title": "{{appTitle}}",
    "description": "{{appDescription}}",
    "resources": "resources.json",
    "crossNavigation": {
      "inbounds": {
        "intent1": {
          "signature": {
            "parameters": {
              "additionalParameters": "allowed"
            },
            "semanticObject": "tutorialFront",
            "action": "display",
            "title": "tutorial"
          }
        }
      }
    }
  }
}
```

* inside the `public` folder of your react app create a `xs-app.json` with the following content
```json
{
  "welcomeFile": "index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^(.*)$",
      "target": "$1",
      "service": "html5-apps-repo-rt",
      "authenticationType": "xsuaa"
    }
  ]
}
```
* inside the `app` folder add the following key to your `package.json`
```json
//...
  "homepage": ".",
//...
```
* and change the script build for the following command, we still don't have the html5-deployer/resources folder, but we'll get there
```json
  "scripts": {
    "build": "BUILD_PATH='../html5-deployer/resources/dist' react-scripts build",
    // ...
  },
```

### HMLT5 Deployer

* Inside the `deploy-react-app-btp-tutorial` folder create a `html5-deployer`
* Inside the `html5-deployer` create a `package.json` file with the following content
```json
{
  "name": "tutorial-html5-deployer",
  "dependencies": {
    "@sap/html5-app-deployer": "2.0.1"
  },
  "scripts": {
    "start": "node node_modules/@sap/html5-app-deployer/index.js"
  }
}
```

* Inside the `html5-deployer` create a `xs-app.json` file with the following content
```json
{
  "welcomeFile": "index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^(.*)$",
      "target": "$1",
      "service": "html5-apps-repo-rt",
      "authenticationType": "xsuaa"
    }
  ]
}
```
* Inside the `html5-deployer` create a `manifest.yaml` file with the following content
```yaml
applications:
  # HTML5 Deployer
  - name: tutorial-html5-deployer
    buildpacks:
      - nodejs_buildpack
    memory: 1024MB
    disk_quota: 1024MB
    no-route: true
    path: .
    services:
      - tutorial-html5-host

```
* Open the terminal inside the `html5-deployer` folder and type the command bellow
```
npm install
```
* Open the terminal inside the `html5-deployer` folder and type the command bellow, this won't work yet

```
cf push tutorial-html5-deployer
```
it'll fail, if you see the logs you'll see this problem
![image](https://user-images.githubusercontent.com/101530871/233164340-1df2c52b-a652-49c8-a3fc-883c6af1ea42.png)

to solve this, we'll go to btp cockpit on Subaccount: trial - Instances and Subscriptions

![image](https://user-images.githubusercontent.com/101530871/233164774-66b785db-742f-4396-8dba-9f22b8439801.png)


and we'll create the bind manually, we can probably do this via cli, but I'm doing the tutorial the way that I did to make it work

![image](https://user-images.githubusercontent.com/101530871/233165089-39e9fda3-8fa6-45f6-a52e-e834e84bd4b1.png)

we'll bind it to the tutorial-html5-deployer
![image](https://user-images.githubusercontent.com/101530871/233165246-c9d8e3ce-efc7-457b-a52d-125f92d5c2f8.png)
![image](https://user-images.githubusercontent.com/101530871/233165449-57a5c4fb-1a07-417c-8107-5367b01dc001.png)

* Open the terminal inside the `html5-deployer` folder and type the command bellow, this time it should work

```
cf push tutorial-html5-deployer
```
You should receive the following response
![image](https://user-images.githubusercontent.com/101530871/233165953-3826f46f-3373-4fb4-afba-9e226bcab254.png)

Now if you go to your approuter URL and make the login you should find your react app :)

![image](https://user-images.githubusercontent.com/101530871/233166618-895fbe9d-9a93-4b56-b0c8-6482a4e17434.png)

I hope this tutorial can help you, I'm open to suggestions and feedback
Have a nice day
