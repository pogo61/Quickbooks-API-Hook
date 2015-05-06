# Akana API HOOK
![Image of Akana] 
(https://www.akana.com/img/formerlyLOGO8.png) 
[Akana.com](http://akana.com)

## Quickbooks API 
### About the API
- report all aspects of your Business via the API
- Home Page: [Quickbooks] (http://quickbooks.intuit.com)
- API Documentation: [Quickbooks API docs] (https://developer.intuit.com/v2/apiexplorer?apiname=V3QBO)

### Pre-Reqs
- you must install the pso extensions custom polices:
    + unzip the com.akana.pso.apihooks.extensions_7.2.0.zip (available in this repository) into the <Policy Manager Home>/sm70 directory. 
    + unzip the com.akana.pso.apihooks.technology.preview_7.2.90.zip (available in this repository) into the <Policy Manager Home>/sm70 directory
    + unzip the com.akana.pso.persistence_7.2.0.zip (available in this repository) into the <Policy Manager Home>/sm70 directory.
    + stop all PM and ND(s)
    + run the configurator in update mode for all the PM and ND instances:
        + run this command, depending on whether you are running on Windows or Linux:
            Windows: 
            [Gateway base dir]\sm70\bin>startup.bat configurator "-Dsilent=true" "-DdeploymentName=Standalone" "-Dproperties=C:/<property file directory location>/myprops.properties" 
     
            UNIX 
            [Gateway base dir]/sm70/bin>startup.sh configurator "-Dsilent=true" "-DdeploymentName=Standalone" "-Dproperties=/export/home/username/<property file directory location>\myprops.properties"
        + the myprops.properties path must be the fully qualified path, and the file contnents will look like:
            container.instance.name=[intance name, e.g. PM]
            credential.username = [administrator login] 
            credential.password = [administrator password] 
            default.host=[instance Host, e.g. localhost] 
            default.port=[instance Port, e.g. 9905]
    + Using the SOA Admin Console, install the following Plug-ins in each PM container:
        * Akana PSO Persistence
    + Using the SOA Admin Console, install the following Plug-ins in each ND container:
        * Akana APIHooks Enhancements
    + restart all PM and ND(s)
- Register yourself and login at [Quickbooks Developer](https://developer.intuit.com)
- Register the application by clicking on the "Create new app" button on the home page.
- Once the App is defined, go to MyApps and click on the App you just defined. Then click the Keys menu item. Record the "OAuth Consumer Key" and "OAuth Consumer Secret"


### Getting Started Instructions
#### Download and Import
- Download QuickbooksAPIHook.zip
- Download the migrations.properties file, and edit it to replace the <replace this with your key> text with the "Container Key" of the ND or ND cluster in your target PM.
    - the container key is found by going to the "Deatils Tab" of the ND cluster, or ND defined in the Policy Manager Console, then looking at the " Container Overview" tab on that page, and copying the "Container Key:" value. ![container key screenshot](https://github.com/pogo61/Google-Sheets-API-Integration/blob/master/Screen%20Shot%202015-03-18%20at%2011.24.45%20am.png "ND Container Key")
- Login to PolicyManager  example: http://localhost:9900
- Select the root "Registry" organisation and click on the "Import Package" from the Actions navigation window on the right side of the screen
  - click on button to browse for the QuickbooksAPIHook.zip archive file 
  - make sure select the migrations.properties file 
  - click Okay to start the importation of the hook.
- this will create a Quickbooks API Hook Organisation with the requisite artefacts needed to run the API.
- you can use the API as is calling http://"URL of the Listener of your ND"/quickbooks_hook, or you can create an API in CM and expost it.
    - if you chose to create an API in CM, you must create a Service in your CM tenant which is a Virtual Service of the Quickbooks_API_Hook_vs0 VS that was created by the import. Then Go to CM and create a API from an existing Service.

#### Verify Import
- Expand the services folder in the Google Sheets API Hook you imported and find Quickbooks_API_Hook_vs0 VS

#### Activate Anonymous Contract
- Expand the contracts folder in the Google Sheets API Hook you imported and find the "Anonymous" contract under the "Provided Contracts" folder
- click on the "Activate Contract" workflow activity in the righ-hand Activities portlet
- ensure that the status changes to "Workflow Is Completed"

#### Configure Security
- Go to Quickbooks API Hook -> Policies -> Operational Policies ->    ProcessVariables policy
    - Click "modify" in the XML Policy Tab. An XML Policy Content editor dialog will be displayed.
    - change the value of the 'redirectURI' element to be the 'Redirect URL' value to https://"ND HOST":"ND HTTPS PORT"/auth_success
    - save the changes
    - click on the "Activate Policy" workflow activity in the righ-hand Activities portlet
    - ensure that the status changes to "State: Active"
- Go to Quickbooks API Hook -> Policies -> Operational Policies -> Insert3-LeggedOAuth1.0Header policy
    - click on the "Activate Policy" workflow activity in the righ-hand Activities portlet
    - ensure that the status changes to "State: Active"

#### Verify Connectivity
- Using 
```curl -X GET  http://ND HOST:ND PORT/quickbooks_hook/helloworld?company_id=1353176045 --header "Content-Type:application/json" --header "authKey:m70tvlms5jj"```

-  the response should be similar to the below:  
    ```{
    "QueryResponse": {
        "Account": [
            {
                "Id": "33",
                "Name": "Accounts Payable (A/P)",
                "sparse": true
            },
            {
                "Id": "84",
                "Name": "Accounts Receivable (A/R)",
                "sparse": true
            },
            {
                "Id": "7",
                "Name": "Advertising",
                "sparse": true
            },
            {
                "Id": "89",
                "Name": "Arizona Dept. of Revenue Payable",
                "sparse": true
            },.......
            {
                "Id": "42",
                "Name": "Visa",
                "sparse": true
            }
        ],
        "maxResults": 91,
        "startPosition": 1
    },
    "time": "2015-05-05T04:14:55.363-07:00"
}```

*Note: the authKey in the curl request, above, is retrieved by using the process in the [MailChimp 3-legged OAuth Client.pdf] (https://github.com/pogo61/MailChimp-API-Hook/blob/master/src/MailChimp%203-legged%20OAuth%20Client.pdf) file in the /src directory*


### How Hello World Works
#### An Akana Integration Primer
The QuickBooks API Hook is a "Virtual Service". That is, its interface is not that of a real service implementation. It can be a proxy to a "real" implementation, or it can be an aggregate (a combination) of a number of "real" implementations. In Policy Manager a "real" implementation is called a "Physical Service".
Apart from offering a different interface to the Physical Service, a Virtual Service offers the ability to attach Policies for security, logging, QoS, and a number of other non-functional capabilities.
Virtual Services also have the ability to have Custom Process and Scripts run before the Physical Service is called. Here is where a lot of the magic of Integration occurs.

#### Hello World
To create the helloworld operation the following was added to a base RAML document to create the [Quickbooks API Helloworld.raml] (https://github.com/pogo61/Quickbooks-API-Hook/blob/master/src/quickbooks%20API%20Helloworld.raml)  document:  
    /helloworld:  
      &nbsp;post:  
        &nbsp;&nbsp;description: "blah blah"  
        &nbsp;&nbsp;&nbsp;responses:  
          &nbsp;&nbsp;&nbsp;&nbsp;200:  
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;body:  
              &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;application/json:  

Then a VS was created by using the RAML as the definition source.
Then the /helloworld Operation in the VS was mapped to the GET /company/{companyId}/query operation in the Quickbooks_API_Hook PS.

Go to the Quickbooks_API_Helloworld VS -> Operations Tab -> GET /hellowworld operation -> Process tab you'll see this image:
![Helloworld process] (https://github.com/pogo61/Quickbooks-API-Hook/blob/master/src/Screen%20Shot%202015-05-05%20at%209.39.56%20pm.png)

Double click on the invoke activity to see how these work to make the Hello World operation call successful.


### Create Your Own Integration with the Google Sheets API
The Hello World operation is one simple way of integrating or extending your API's.
Take a look at the [Quickbooks API Integration](https://github.com/pogo61/MailChimp-API-Integration). This will give you a deeper inderstanding of the richness of our gateway product in integrating to API's    

### Modify and Build
In the event you need to change the API Hook.   Here are the instructions to do so. 

### License
Put a link to an open source license

