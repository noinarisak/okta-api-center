This how-to article shows you how to protect REST APIs using the Azure API Management (APIM) service, by using Okta as the OpenID Connect provider.

Prerequisites
To follow the steps in this article, you must have:

an API (can be deployed in Azure or else where)
an Azure account
a Okta tenant 
Instructions
Here is a quick overview of the steps:

Create an Azure API Management (APIM) Instance.
Add an API for Azure APIM to manage 
Register Okta as the OpenID Connect Provider
Add the validate-jwt policy to verify the access tokens passed by an API client
Test the APIs using Postman


Create an Azure API Management (APIM) Instance
In your Azure instance, create a new API Management instance



Provide details like name, region and pricing details. In this article, we have created chevron-poc with the region as 'US West 2'



Click Create. This will take a few minutes to deploy and activate.



Add an API for APIM to Manage
Click on APIs in the left navigation pane and click '+ Add API'

In this article, we have added an API using 'OpenAPI'. https://chevrontodoapi.azurewebsites.net/swagger/v1/swagger.json





After the API is registered, Go to the settings section for 'Chevron Todo API' and add more details. Uncheck the 'Subscription required' checkbox.



Currently the API is unprotected, in the next few steps we will configure Okta as the OpenID Connect Provider and add rules in APIM to validate the access tokens.

Register Okta as the OpenID Connect Provider
In the Okta Admin dashboard, create an OpenID Connect Web Application. Set a placeholder Login redirect URI like https://placeholder/ for the interim.



Back in Azure APIM, Go to Security → OpenID Connect in the left navigation pane. Click on '+' to register a new OpenID Connect Provider.

In this article, we have used the default Authorization Server's OpenID connect provider: https://chevron-poc.oktapreview.com/oauth2/default/.well-known/openid-configuration

Also specify the Client ID and Client Secret that was created in the earlier step.



Copy the generated redirect_uri from APIM and paste in Okta App Login Redirect URI.







Next, we will add the 'validate-jwt' rule to check for a valid access token.

Add Validate JWT Rule
Go back to API → Chevron Todo API → Security

Switch the User authorization from None to 'OpenID connect'. Select the provider your registered above 'Okta Default OpenID Connect'.











 Go to the Design tab and click 'Add policy' for 'Inbound processing' for all API operations.



Select 'Validate JWT' (validate-jwt)



Configure the JWT rule:

Look for Authorization header
Check for audience and issuer information
Validate JWT according to the openid-configuration

```xml
<policies>
    <inbound>
        <base />
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Access Token missing or invalid" require-expiration-time="true" require-scheme="Bearer" require-signed-tokens="true" clock-skew="10">
            <openid-config url="https://chevron-poc.oktapreview.com/oauth2/default/.well-known/openid-configuration" />
            <audiences>
                <audience>api://default</audience>
            </audiences>
            <issuers>
                <issuer>https://chevron-poc.oktapreview.com/oauth2/default</issuer>
            </issuers>
        </validate-jwt>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

Testing the APIs using Postman


The Base APIs are available at: https://chevrontodoapi.azurewebsites.net/swagger/index.html

Using Postman send a GET request to https://chevrontodoapi.azurewebsites.net/api/TodoItems 

This returns [] 

Next, change the endpoint to the APIM endpoint: https://chevron-poc.azure-api.net/api/TodoItems 

Set the Auth for the request to OAuth 2.0

Add auth data to Request Headers

Click on Get New Access Token

Enter Details for the OAuth2 Client App



Click Request Token, this launches a browser and initiates the authorization code flow for the APIM callback URL

Click Use Token and Send the Request

If the token is correctly validated, you should see the same response as to the base API.

If the token is not validated, you will get this error message:

`{

"statusCode": 401,
"message": "Access Token missing or invalid"
}`
To diagnose the jwt validation, go back to APIM, click on Overview and look for 'Gateway Errors'
The JWT related errors begin with Token -  TokenNotPresent, TokenInvalidAudience, TokenSignatureKeyNotFound, ...

Now, you have an API protected with Okta.