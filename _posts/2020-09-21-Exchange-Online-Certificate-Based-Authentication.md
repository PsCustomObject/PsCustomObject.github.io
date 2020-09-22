---
title: "Exchange Online - Certificate Based Authentication - Step by Step"
excerpt: "Microsoft is deprecating basic authentication when establishing a connection to Exchange Online in favor of more secure Certificate Based Authentication (CBA). In this step by step article we will explore how to configure all the components from scratch"
categories:
  - PowerShell
  - Office365
  - Exchange
tags:
  - PowerShell
  - Office365
  - Exchange Online
  - Exchange Online CBA
  - Exchange Online Certificate Based Authentication
  - EXO CBA
  - PowerShell Exchange Certificate

toc: true
header:
    teaser: "/assets/images/ExchangeLogo.png"
---

Microsoft released to GA the new version of *Exchange Online Management* module, version *2.0.3* at the time this article, which introduces **Certificate Based Authentication** for PowerShell sessions. *Basic Authentication* has been already deprecated and originally planned for removal in *October 2020* but due CoVid-19 outbreak this has been postponed to 2021 as you can [read here](https://techcommunity.microsoft.com/t5/exchange-team-blog/basic-authentication-and-exchange-online-april-2020-update/ba-p/1275508).

I have been using the module preview in production for quite some time but held back publishing this article so to have all places in place as GA.

## Exchange Online Certificate Based authentication - Register Azure Application

The first step to deploy Certificate Based authentication is to register a new *Azure Application*. Navigate *Azure Active Directory* in the Azure portal and select **App Registrations** (alternatively use the search function which is what I usually do)

Testing clickable images:

<figure>
<img src="/assets/images/Azure_Application_Registration_Blade.png"></a>
</figure>

![Azure Application Registration](/assets/images/Azure_Application_Registration_Blade.png)

In the app registrations under *Owned applications* we can list all applications that we registered under our account, in my case this is still empty, and under *All applications* as the name implies all application registered tenant wide.

![Azure owned applications](/assets/images/Azure_Owned_Applications.png)

Click on the **New Registration** and fill the various fields accordingly, unless you have specific needs all default values should suffice for most configurations/deployments. I am assuming a single tenant deployment in the following example

![EXO Certificate Authentication Application](/assets/images/ExoV2_Registration.png)

Once done click on the **Register** button, provision will only take a couple of seconds.

![EXO Certificate Authentication Application](/assets/images/Azure_Application_Registration_Summary.png)

**Note:** The only field really needed in the *Name* one just be sure to chose a descriptive name that is easy for you to remember.
{: .notice--danger}

### Exchange Online Certificate Based authentication - Grant API Permissions

Once the application has been registered we need to configure/grant API permissions that will define what our application can and cannot do. Either select *API Permissions* from the left blade or from the link directly below the API properties and select **Add permission**

![ExOV2 Certificate Authentication Application](/assets/images/Configure_API_Permissions.png)

From the **Request API Permissions** scroll all the way down the *Supported Legacy APIs* and select **Exchange**, **Application Permissions** and finally tick **Exchange.ManageAsApp** under the *Exchange* section

**Note:** I will not go into much detail as much has already been written about this but Exchange does not natively support new *Graph* API that's why Exchange is listed under *Legacy API*.
{: .notice--primary}

![ExOV2 Certificate Authentication Application](/assets/images/Exchange_API_Permissions.png)
![ExOV2 Certificate Authentication Application](/assets/images/Exchange_API_Permissions._2.png)

The last step involves clicking the **Grant Admin Consent for <your tenant name>** so that permissions will be deployed for all mailboxes.

![ExOV2 Certificate Authentication Application](/assets/images/EXO_Grant_Admin_Consent.png)

## Exchange Online Certificate Based authentication - Configure Authentication

With the application created configured in AzureAD we need to configure authentication against AzureAD. When using *[application permissions model](https://en.wikipedia.org/wiki/Application_permissions)* authentication is performed via a **client secret**, a token, or a **certificate**. Token authentication is considered, rightly so, *less secure* for this reason only certificate one is supported by Exchange Online/Microsoft.

In the scope of Exchange OnLine authentication it is unimportant if we're using a self signed or publicly trusted certificate as long as **we have the associated private key**.

You can easily create a self-signed certificate via PowerShell with the following command:

```powershell
$paramNewSelfSignedCertificate = @{
    Subject           = 'Exchange Online Background Process'
    CertStoreLocation = 'cert:\CurrentUser\My'
    KeySpec           = 'KeyExchange'
    FriendlyName      = 'Exchange Online Authentication Certificate'
}

New-SelfSignedCertificate @paramNewSelfSignedCertificate

# Output
Thumbprint                                Subject
----------                                -------
1B1541D37888EFECD058B528524764G0EF8608D4  CN=Exchange Online Background Process
```

**Note:** Certificate will be automatically installed in the local certificate store under *User / Personal Certificates* and have, by default, validity of 1 year.
{: .notice--primary}

Open *Certificate Manager* MMC console and under *Certificates Current User / Personal / Certificates* right-click on the certificate and select *All Tasks / Export*. Just follow the export wizard be sure to select *Do not export private key* and select **CER** as the export format.

In the Azure Portal select *Certificates and Secretes* from the left blade and *Upload certificate* navigating to the path where the certificate has been expoted/stored

![ExOV2 Certificate Authentication Certificate Upload](/assets/images/Azure_Certificate_Upload.png)

**Note:** Write down the certificate thumbprint displayed in the Azure page as we will need this later on.
{: .notice--primary}

## Exchange Online Certificate Based authentication - Grant permissions

As I mentioned in the **Grant API Permissions** paragraph Graph API does not support any Exchange management operations nor we can use Exchange **RBAC** model as that only applies to user objects not applications, like in our case, which are represented by a **Service Principal**.
What we can do is granting a **[AzureAD Directory Role](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles#exchange-service-administrator-permissions)** to our application Service Principal.

With the *Azure AD* blade selected go to **Roles and administrators** and select **Exchange Administrator** confirming with the *Add Assignment* button

![ExOV2 Certificate Authentication Role Assignment](/assets/images/Azure_Exchange_Admin_Role.png)

In *Select Member* windows you will need to search application by GUID and select it

![Application Role Assignment](/assets/images/Azure_Exchange_Assignment.png)

In the *Add Assignment* page be sure to select **Active** under *Assignment Type* and tick the **Permanently Assign** checkbox

![Assignment Page](/assets/images/Add_Assignment_Page.png)

Once configuration is complete you will see a page similar the following

![Assignment Confirmation](/assets/images/Azure_Assignment_Confirmation.png) where you can review applied configuration and make any required change.

## Exchange Online Certificate Based authentication - Testing connection

After all the above components are in place it's time to test our configuration. Assuming the *Exchange Online Management* PowerShell module version 2.0.3 is installed we can issue the following command to establish a connection

```powershell
$paramConnectExchangeOnline = @{
    CertificateThumbprint = $certThumbPrint
    AppId                 = $appId
    Organization          = $exchangeOrgId
}

Connect-ExchangeOnline @paramConnectExchangeOnline
```

Where *certThumbPrint* is the certificate thumbprint we created and uploaded to Azure, *appId* is the ID of the application we created and *exchangeOrgId* is the name of the tenant in the form **tenantName.onmicrosoft.com**

And here's the result

![Get Mailbox result](/assets/images/Exchange_Get_Mailbox.png)

## Closing notes

This was quite a long post but steps to get up and running with Exchange Online Certificate based authentication are numerous even if not difficult to implement but well worth following.

Certificate Based authentication resolves a number of challenges administrators had to face up to this point, chief among all [storing credentials](https://pscustomobject.github.io/powershell/howto/Store-Credentials-in-PowerShell-Script/) which is inherently insecure.
