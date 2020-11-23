# Dev_Eri_Azure_AD_B2C
.NET Core 3.1 MVC with razor integrate with Azure AD B2C and use Microsoft Graph API for custom attribute

Purpose<br>
To manage Azure B2C for user logins to registered applications<br>

Procedure<br>
Switch directory containing azure ad b2c<br>

Search and open azure ad b2c<br>
Choose register new application<br>
Add new application<br>
Fill in display name (name for application)<br>
For Include web app/ web API and Allow implicit flow, select Yes.<br>
Fill in Reply URI <br>
Click Register<br>
Create client secret<br>
In the Azure AD B2C - Applications page, select the application you created<br>
Select Keys and then select Generate key.<br>
Select Save to view the key. Make note of the App key value. You use this value as the application secret in your application's code.(We will use this when we use custom attribute and calling it using microsoft graph)<br>
Add new user flow<br>
Under Policies, select User flows (policies), and then select New user flow.<br>
On the Recommended tab, select the Sign up and sign in user flow.<br>
Enter name for policy<br>
For Identity providers, select Email signup<br>
For User attributes and claims, choose the claims and attributes that you want to collect and send from the user during sign-up.<br>
Click create <br><br>
Add authentication codes to application<br>
There are lot of ways of doing this. This step is using dependency injection in configure services(startup.cs)<br>
In Appsettings.json here is the configuration the example of configuration<br>
"AzureAdB2C": {<br>
    "Instance": "https://yourcompany.b2clogin.com/tfp/",<br>
    "ClientId": "922gag04-eb29-480a-a664-1ee73b3c0863",<br>
    "CallbackPath": "/signin-oidc", <br>
    "Domain": "yourcompany.onmicrosoft.com",<br>
    "SignUpSignInPolicyId": "B2C_1_eri_test_net_core",<br>
    "ResetPasswordPolicyId": "B2C_1_passwordreset1",<br>
    "EditProfilePolicyId": "B2C_1_profileediting1"<br>
  }<br>
Callbackpath must must with reply uri we set previously. Default is using /signin-oidc<br>
In startup.cs(Configureservices function) add this code in first line<br>
services.AddAuthentication(AzureADB2CDefaults.AuthenticationScheme)<br>
                .AddAzureADB2C(options => Configuration.Bind("AzureAdB2C", options));<br>
In startup.cs(Configure function), add this code before app.UseEndPoints<br>
app.UseAuthentication();<br>
To Call sign in, sign out and edit profile, can use this example(This is .NET Core 3.1 code)<br>
Basically it’s redirecting to some URL like  /AzureADB2C/Account/SignIn and in the cloud it will point to our sign in page. <br>
Add authorization codes to application<br>
This is a workaround because azure ad b2c doesn’t provide group management. So what we do is creating custom attribute(e.g: AppRole) then using this we bind to policy
<br><br>
Create custom attribute<br>
In Azure ad b2c, select user attributes<br>
Click add<br>
Enter name, choose data type and enter description<br>
Click create<br>

Enable Permission<br>
Select App registrations, and then select Application we want to authorize.<br>
Under Manage, select API permissions.<br>
Under Configured permissions, select Add a permission.<br>
Select the Microsoft APIs tab, then select Microsoft Graph.<br>
Select Application permissions. Choose User.Read.All, User.ReadWrite.All, Directory.Read.All, Directory.ReadWrite.All<br>
Select Add permissions. As directed, wait a few minutes before proceeding to the next step.<br>
Select Grant admin consent for (your tenant name).<br>
Select your currently signed-in administrator account, or sign in with an account in your Azure AD B2C tenant that's been assigned at least the Cloud application administrator role.<br>
Select Accept.<br>
Select Refresh, and then verify that "Granted for ..." appears under Status. It might take a few minutes for the permissions to propagate.<br>
Update custom attribute<br>
Remember to add custom attributes in application claims in sign up and sing in flow. We use postman to update custom attribute<br>
Before we can update we need to have an access token. Here is how to get access token<br>
URL is https://login.microsoftonline.com/{tenant}/oauth2/token<br>
Client id is The Application ID that the registration portal) assigned to your app.<br>
Client secret is The application secret that you created in the app registration portal for your app. It should not be used in a native app, because client_secrets cannot be reliably stored on devices. It is required for web apps and web APIs, which have the ability to store the client_secret securely on the server side.<br>
Resource and grant type use it like that.<br>
After getting the access token, we will use the token to update, first we will get all user to get the user id(we will use this id to update the specific user)<br>
Call the api from postman with method PATCH. The URL is<br>
 https://graph.microsoft.com/v1.0/users/{userid}<br>
For the param, use format like this<br>
extensioin_{applicationId}_{customAttributeName<br>
{<br>
    "extension_yy0e6ec3b4274b57ab9a83632f79681b_applicationRole": "tes-app-role"<br>
}<br>
ApplicationId in here is from b2c-extensions-app application id without “-”<br>
If success will return 204 no content<br>
To check user have this attribute can call microsoft graph using<br>
this syntax(normal return won’t show extension).<br>
https://graph.microsoft.com/v1.0/users/{userid}?$select=extensioin_{applicationId}_{customAttributeName<br>
