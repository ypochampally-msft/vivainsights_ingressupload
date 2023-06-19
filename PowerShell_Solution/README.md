# PowerShell script to run the Descriptive Data Upload Api 


## Install MSAL.PS
Use the command below or go to https://www.powershellgallery.com/packages/MSAL.PS for instructions on installation

``` Install-Module -Name MSAL.PS ```



## Inputs parameters for the script 

Mandatory parameters 
1.	App (client) ID. Find this ID in the registered app information on the Azure portal under **Application (client) ID**. If you haven’t created and registered your app yet, follow the instructions in our main data import documentation, under Register a new app in Azure.
2.	Path to the zip folder. Format the path like this: `C:\\Users\\JaneDoe\\OneDrive - Microsoft\\Desktop\\info.zip`. If you haven't downloaded the zip folder yet, find it [here](https://microsoft-my.sharepoint.com/:f:/p/ypochampally/EgrLeWo6hBJNiZdSbmrSA2gB5MgmZGXtuBy7Z-0tZ91NZw?e=X1O5o2)  . Refer to our [main data-import documentation](https://learn.microsoft.com/viva/insights/advanced/admin/import-org-data-first#prepare-the-data-export) for more information about using the files in this folder. The file names data.csv and meta.json should not be changed
3.	Azure Active Directory tenant ID. Also find this ID on the app's overview page under **Directory (tenant) ID**.
4.	Scale Unit: The scaleunit for your teanant. Eg: novaprdwus2-02  
5.  Connector Type: `HR` or `Survey`
6.  Optional parameters: Either the certificate name or the client secret needs to be provided 
7.	Certificate name: This name is configured in your registered application. If you haven’t created a certificate yet, refer to [How to create a self-signed certificate](https://learn.microsoft.com/azure/active-directory/develop/howto-create-self-signed-certificate). After you upload the certificate, the certificate name shows up under **Description** in the Azure Portal.
8.	Client secret: A secret string that the application uses to prove its identity when requesting a token. Also can be referred to as application password. This is only shown for the first time when the client secret is created. 

Examples

1. ``` .\DescriptiveDataUpload.ps1 -ClientId **** -pathToZippedFile  "C:\repos\temp\info.zip" -TenantId ***** -novaScaleUnit novappewus2-02 -connectorType HR -ClientSecret ****```

2. ``` .\DescriptiveDataUpload.ps1 -ClientId **** -pathToZippedFile  "C:\repos\temp\info.zip" -TenantId ***** -novaScaleUnit novappewus2-02 -connectorType Survey -certificateName CN=ypochampally-certificate```

## API that the script runs 

Sample request:
``` 
 Method: POST, RequestUri: 'https://api.orginsights.viva.office.com/v1.0/scopes/<tenantId>/ingress/conne
                      ctors/HR/ingestions/fileIngestion', Version: 1.1, Content: System.Net.Http.MultipartFormDataContent, Headers:
                      {
                        Accept: application/json
                        x-nova-scaleunit: novaprdwus2-02
                        Authorization: Bearer <bearer token from ClientID and client certificate/secret>"
                        Content-Length: 729
                      }
                      Content: <zip file>
``` 
Sample response: 
```
 {"FriendlyName":"Data ingress","Id":"<ingestion Id>","ConnectorId":"<connector Id>","Submitter":"System","StartDate":"2023-05-08T19:07:07.4994043Z","Status":"NotStarted","ErrorDetail":null,"EndDate":null,"Type":"FileIngestion"}
```

**Possible errors in the API**
1. Missing header <Authorization>: Response status 403 
2. Missing zip file: Response status 500
3. Expired Authorization header: Response status 200, but the response will suggest to sign in again 
4. If the zip file is corrupt: the response status may be 200, but the ingestion will be stuck in the "NotStarted" state  
5. If the filenames in the zip - data.csv and meta.json are changed: the response may be 200 , but the ingestion will be stuck in the "ExtractionCompleted" state
6. If the meta.json and data.csv have incorrect data (i.e mismatch in the fieldnames/number of fields, empty fields, etc ): the response status will be 200, but the ingestion will be stuck in "ValidationFailed" state

