# Dev_Eri_Azure_AD_B2C
.NET Core 3.1 MVC with razor integrate with Azure AD B2C and use Microsoft Graph API for custom attribute

Purpose
To manage Azure B2C for user logins to registered applications

Procedure
Switch directory containing azure ad b2c

Search and open azure ad b2c
Choose register new application
Add new application
Fill in display name (name for application)
For Include web app/ web API and Allow implicit flow, select Yes.
Fill in Reply URI 
Click Register
Create client secret
In the Azure AD B2C - Applications page, select the application you created
Select Keys and then select Generate key.
Select Save to view the key. Make note of the App key value. You use this value as the application secret in your application's code.(We will use this when we use custom attribute and calling it using microsoft graph)
Add new user flow
Under Policies, select User flows (policies), and then select New user flow.
On the Recommended tab, select the Sign up and sign in user flow.
Enter name for policy
For Identity providers, select Email signup
For User attributes and claims, choose the claims and attributes that you want to collect and send from the user during sign-up.
Click create 
Add new user login / Invite user to login


Add authentication codes to application
There are lot of ways of doing this. This step is using dependency injection in configure services(startup.cs)
In Appsettings.json here is the configuration the example of configuration
"AzureAdB2C": {
    "Instance": "https://yourcompany.b2clogin.com/tfp/",
    "ClientId": "922gag04-eb29-480a-a664-1ee73b3c0863",
    "CallbackPath": "/signin-oidc", 
    "Domain": "yourcompany.onmicrosoft.com",
    "SignUpSignInPolicyId": "B2C_1_eri_test_net_core",
    "ResetPasswordPolicyId": "B2C_1_passwordreset1",
    "EditProfilePolicyId": "B2C_1_profileediting1"
  }
Callbackpath must must with reply uri we set previously. Default is using /signin-oidc
In startup.cs(Configureservices function) add this code in first line
services.AddAuthentication(AzureADB2CDefaults.AuthenticationScheme)
                .AddAzureADB2C(options => Configuration.Bind("AzureAdB2C", options));


In startup.cs(Configure function), add this code before app.UseEndPoints
app.UseAuthentication();
To Call sign in, sign out and edit profile, can use this example(This is .NET Core 3.1 code)

Basically it’s redirecting to some URL like  /AzureADB2C/Account/SignIn and in the cloud it will point to our sign in page. 
Add authorization codes to application
This is a workaround because azure ad b2c doesn’t provide group management. So what we do is creating custom attribute(e.g: AppRole) then using this we bind to policy

Create custom attribute
In Azure ad b2c, select user attributes
Click add
Enter name, choose data type and enter description
Click create
Enable Permission
Select App registrations, and then select Application we want to authorize.
Under Manage, select API permissions.
Under Configured permissions, select Add a permission.
Select the Microsoft APIs tab, then select Microsoft Graph.
Select Application permissions. Choose User.Read.All, User.ReadWrite.All, Directory.Read.All, Directory.ReadWrite.All
Select Add permissions. As directed, wait a few minutes before proceeding to the next step.
Select Grant admin consent for (your tenant name).
Select your currently signed-in administrator account, or sign in with an account in your Azure AD B2C tenant that's been assigned at least the Cloud application administrator role.
Select Accept.
Select Refresh, and then verify that "Granted for ..." appears under Status. It might take a few minutes for the permissions to propagate.
Update custom attribute
Remember to add custom attributes in application claims in sign up and sing in flow. We use postman to update custom attribute
Before we can update we need to have an access token. Here is how to get access token
URL is https://login.microsoftonline.com/{tenant}/oauth2/token
Client id is The Application ID that the registration portal) assigned to your app.
Client secret is The application secret that you created in the app registration portal for your app. It should not be used in a native app, because client_secrets cannot be reliably stored on devices. It is required for web apps and web APIs, which have the ability to store the client_secret securely on the server side.
Resource and grant type use it like that.

After getting the access token, we will use the token to update, first we will get all user to get the user id(we will use this id to update the specific user)
Call the api from postman with method PATCH. The URL is
 https://graph.microsoft.com/v1.0/users/{userid}
For the param, use format like this
extensioin_{applicationId}_{customAttributeName
{
    "extension_yy0e6ec3b4274b57ab9a83632f79681b_applicationRole": "tes-app-role"
}

ApplicationId in here is from b2c-extensions-app application id without “-”

If success will return 204 no content
To check user have this attribute can call microsoft graph using
this syntax(normal return won’t show extension).
https://graph.microsoft.com/v1.0/users/{userid}?$select=extensioin_{applicationId}_{customAttributeName
