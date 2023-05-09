# aws-cognito-api

This tutorial is made to explain how to configure an AWS Cognito with AWS API.

## Create AWS DynamoDB table

Go to DynamoDB and create a new table called "Items" with partition key named "id" with type string, then go into additional information in general tab and copy the Amazon Resource Number (ARN) value (arn:aws:dynamodb:us-east-1:XXXXXXXXXXXX:table/Items).

## Create Lambda Function

In your AWS account, go to Lambda Function and create a new function called "items_put_function" in Nodejs 14.x. Then copy and paste the code in this repository (the content of the file "index.js") at the code tab and replace the content inside the index.js file in Amazon Lambda. Once done, go to Configuration tab > then permissions and click on the link below the role name. Once there, still in the permissions tab, go to "Add Permissions" and "Add inline policy". Then add the following:

- Service: DynamoDB
- Action: Write > putItem
- Resource: Add ARN and paste the ARN value from your previously created DynamoDB.
- No condition access.

Then click on Review Policy and create.

## Create API Gateway

Still logged in your AWS account, go to API Gateway and create a new one at upper right. Select API Rest (non private). Leave Protocol to "REST" and select "Choose new API". Then name your API as "item_api". Once created, go to "Resources" and in Actions button, select "Create Resource", name it as items and select "create". Then select your newly "/items" resource on the list, and again in Actions button, select "Create Method" and select POST.

Then select "POST" and leave "Lambda Function" selected as integration type. Select "Use Lambda Proxy Integration" and write your Lambda function (items_puts_function) in "Lambda Function" field then click save.

After done that go again to Actions button, this time select "Deploy API" and do the following:

- Deployment Stage: [new stage] 
- Deployment name: any name you want, it's preferable to put "dev"
- Stage description: leave blank
- Deployment description: leave blank

Click deploy. You will see in "Stages" tab the invoke URL value, copy that and open Postman or any other application you prefer to test HTTP requests and concat that URL value with the endpoint you just created (/items). Make the Method POST and enable body as JSON and paste the following:

```
{
    "id": "400",
    "price": 6000
}
``` 

Test it out and you should see a message like this: "Item successfully added!". Check your DynamoDB table to see the new item added.

## Create a User Pool for authentication.

Once finished setting up the whole API inserting on DynamoDB. Type "Cognito" on AWS search bar and create a UserPool with the following informations:

### Page 1

Leave everything default and select "email" on login options. Then proceed to the next page.

### Page 2

(Optional) leave MFA as none. And unless you want to customize password and password recovery, leave everything as be.

### Page 3

Leave everything as be.

### Page 4

- Select e-mail provider as "Send Cognito e-mails."

### Page 5

- Name your user pool as "UserPool"
- Select "Use Cognito's provider user interface"
- On Cognito's domain: (Create a unique name for your domain)
- On client application name's, put "AppClient"
- Select "Generate client's secret"
- URL Callback: http://example.com

Inside Advanced Client Application Settings:

- ALLOW_CUSTOM_AUTH
- ALLOW_USER_SRP_AUTH
- Identity Providers: Cognito User Pool
- OAuth 2.0 Concession type: Authorization code; Implicit
- Scopes: OpenID, E-mail.

Leave the rest as default.

### Page 6

Review your actions and create at the end.

### Configuring your API to use the UserPool Authorizer

Once you created de UserPool and AppClient, go back to API Gateway and go to "Authorizers" tab, create new Authorizer and fill the following:

- Name: ItemAuthorizer
- Type: Cognito
- User Pool: select your pool
- Token Source: Authorization

Create.

Go back to "Resources" > "POST" (on /items) and click on Method Request. Select "ItemAuthorizer" and click on the check button next to the field.

### Setting up the Cognito Account

With Postman or any other API program open, create a new collection and on "Authorization" tab put the following:

- Type: OAuth 2.0
- Add auth data to: Request Headers
- Token: Available Tokens
- Header Prefix: Bearer
- Token name: Token
- Grant Type: Implicit
- Callback URL: https://example.com
- Auth URL: https://(unique name for your domain).auth.us-east-1.amazoncognito.com/login
- Client ID: (ClientID from AppClient)
- Scope: email openID
- Client Authentication:  Send client credentials in body

Then click on "Get New Access Token", create a new account, validate email, login and you should see the JWTs generated for email and openID scopes and done!

## Testing the application

With the collection saved, place or create a new HTTP Request inside your collection, and make sure the request's type is "Inherit auth from collection". With the HTTP request ready ( the same from API Gateway section), change the JSON body values to your liking and hit send, and the application will still be working!
