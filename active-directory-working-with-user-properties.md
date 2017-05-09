---
title: Working with user properties in Azure Active Directory | Microsoft Docs
description: Explains how to work with user properties in Azure Active Directory - what are they used for, how to import, create, export, use in SaaS applications.
services: active-directory
documentationcenter: ''
author: rodejo
manager: ''
editor: ''

ms.assetid:
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/04/2017
ms.author: rodejo

---
# Working with user properties in azure Active Directory
This article explains about how you can work with user properties in Azure Active Directory.

## What are user properties?
User properties are attributes of a user object that an be assigned values. You would normally use these values to distinguish user objects from each other, such as when you assign a value to user object's DisplayName property, or to use the value in other subsystems, such as the user's Authentication Phone, which is used to notify a user of a password reset requests. Another common use for user properties is to configure Azure Active Directory to provision a user's properties into a target SaaS application. We'll look at an example of how to configure this later in this article.
Azure Active Directory contains tow different classes of user properties:

1. Built-in properties. These are the most commonly used properties. You can find the list of Azure Active Directory built-in properties [here](Link to Azure AD built-in properties). Note that some of these properties are created by the system and cannot be modified by the administrator.
2. Extension properties. Extension properties are created to hold values for properties that do not exist as built-in properties. Extension properties are automatically created by Azure Active Directory Connect when you enable [Directory Extension Attribute Sync](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnectsync-feature-directory-extensions). Directory extensions allows you to extend the schema in Azure Active Directory with your own properties from on-premises Active Directory. These attributes can either be pre-existing properties that exists in Active Directory but do not exist in in Azure Active Directory, or these could be properties that you created yourself in the Active Directory Schema - you can read more about how to do that [here](https://msdn.microsoft.com/en-us/library/ms676900.aspx). Note that you can also create your own custom extension properties without Azure Active Directory Connect using PowerShell - more about how to do this can be found in [this article](https://docs.microsoft.com/en-us/powershell/azure/using-extension-attributes-sample?view=azureadps-2.0).
  
## What can I do with user properties?

Many features of Azure Active Directory use user properties. Here are some of the things you can do with user properties in Azure Active Directory:

+ Use a user's email address or phone number to enable password reset verification and multi-factor authentication;
+ Show user properties on the Profile Page in the Access Panel;
+ Show a user's thumbnail photo as a visual cue for a user's object in the Admin Portal; 
+ Use user properties in dynamic groups to manage a user's group memberships. When you assign these dynamic groups to provide access to resources such as Applications, Licenses or SharePoint folders you can implement Attribute Based Access Control to these resources;
+ Search for users or sort users in the Admin Portal;
+ Provision user properties in connected SaaS applications and in other Microsoft services such as SharePoint, Skype and Exchange; 
+ Configure SAML tokens to (provide user property values as claims)[https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization] to SAML-enabled applications.

and next to the items in this list, you can also use user properties in your own applications that call Microsoft Graph or use Azure Active Directory PowerShell, or export users and user properties to be used in other applications. 

## How can I populate user properties?

There are three typical methods to populate the value of a user's properties:

1. Synchronize the value from on premises Active Directory. This is the most commonly used method to populate your directory with users and user property values, and you would usually use Azure Active directory connect for this. We'll discuss this in more detail below.
2. Manually set the value for a user's properties through the Azure Active Directory Portal. As an administrator, you can see and modify values of user properties using the User page in the portal. Note that not all properties can be changed - properties who's values are being synced from on premises Active Directory cannot bet changed in Azure Active Directory.
3. Using PowerShell. You can create new users and set user property values using PowerShell cmdlets. You can also write scripts that import user objects and their properties from an external source such as a CSV-file or update user properties from an external source.

When using PowerShell or Azure Active Directory Connect to create or update users in Azure Active Directory you can optionally transform values of properties - for example, you could create a new value for a user's DisplayName by combining the values for FirstName, LastName and Department - so now John Snow who works in the Securities department would have a display name of "John Snow (Securities)".  

### Populating user properties using PowerShell

You can populate a user's built-in property values using the Set-AzureADUser cmdlet. An example for how to set a user's City property is 

```powershell
Set-AzureADUser -ObjectId AbbieS@WoodGroveOnline.com -City "New York"
```

Note that you can only modify property values for users in Azure Active Directory for those properties that are not being synced from on premises Active Directory. If you need to modify the property values for properties that are synced form on premises Active Directory you need to modify the property values in Active Directory, the values will then be synced to Azure Active Directory through Azure Active Directory Connect.

## How can I create new user properties in Azure Active Directory?

The Azure Active Directory schema comes with a wealth of user properties that can be used to provide support for the scenarios you need to implement. However, most customers have e need for specific properties that are not part of the built-in schema of Azure Active Directory. There are three ways you can provide support for properties that are not part of the schema:

1. Use extensionAttributes - on premises Active Directory supports several different classes of extensionattributes - these are predefined but unused properties that you can use to store data as you see fit. When you sync these properties to Azure Active Directory, the Azure Active Directory schema is automatically extended with these properties and user property values are synced.
2. Create new user properties in Active Directory and sync them to Azure Active Directory. Just like with extensionAttributes, Azure Active Directory COnnect will extend the Azure Active Directory schema to include these new properties and user property values will be synced.
3. Create you own custom Azure Active Directory schema extensions. Note that Azure Active Directory connect will not sync any data to schema extension properties you created yourself - if you want to populate these properties you would either use Microsoft Graph or PowerShell.

### Should I create new extension properties or use existing extensionAttribute properties in on premises Active Directory? 

On premises Active Directory comes with a number of extension properties that you can use to store data. These are:

+ extensionAttribute1 - extensionAttribute15
+ msDS-cloudExtensionAttribute1 - msDS-cloudExtensionAttribute20 

If you installed Exchange Server in your Active Directory domain or extended the Active Directory schema with the Exchange schema extension you will also see:

+ msExchExtensionAttribute16 - msExchExtensionAttribute45
+ msExchExtensionCustomAttribute1 - msExchExtensionCustomAttribute5 (note that these are multi-valued string properties)

You can use these properties to store data and sync this data to Azure Active Directory. Note that you will need to enable these attributes to be synced to Azure Active Directory using Azure Active Directory Connect and configure [Directory Extension Attribute Sync](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnectsync-feature-directory-extensions). 

If the existing properties in Active Directory do not fit with the type of data you want to store, you should extend the schema. It is important to note that schema additions in Active Directory are permanent; you can disable properties, but you can never remove them from the schema. Keep this in mind when you are testing code.
Also consider the size of the data that you want to store. Microsoft recommends that no property value exceed 500 kilobytes, including the sum of multivalued properties. Also, objects should not exceed 1 megabyte in size. Consider also the number of instances of the data; if you add a new property to the User class on a system that has 100,000 users, this can use up considerable space.

## Syncing user properties from on premises Active Directory to Azure Active Directory 

### What is the default set of properties that get synced from on premises Active Directory to Azure Active Directory?



### How can I prevent user properties from getting synced to Azure Active Directory?

### What is the minimum set of user properties that must get synced to Azure Active Directory?

### How can I add user properties to get synced to Azure Active Directory?

### What are considerations when adding new user properties to be synced to Azure Active Directory?

#### What are best practices for syncing properties?	

#### Which properties should I never sync?

### How can I remove synced properties and their values for Azure Active Directory?

## How do I provision user properties to a SaaS application?

### Which user properties should I provision?

### Step-by-step example: How to configure Azure Active Directory so I can sync users from on premises Active Directory to ServiceNow

This paragraph shows which steps you must follow to configure Azure Active Directory so that your users in on premises Active Directory can sign in into and use the ServiceNow SaaS application. 

ServiceNow is a software platform that supports IT service management and automates common business processes. It is one of the most popular Software-as-a-Service applications that is enabled in the Azure Active Directory application platform and we'll use it as an example of how to configure Azure Active Directory so that your users are automatically provisioned and can sign in into ServiceNow, enabling the controls in Azure Active Directory for application access control and credentials management.

For users to be able to sign in and use ServiceNow, several steps have to be followed:

1. We need to make sure the user information ServiceNow needs is available in on premises Active Directory
2. We must configure Azure Active Directory connect so that this information gets synchronized with Azure Active Directory
3. We need to configure an application object for the ServiceNow SaaS app. the application object is used to store the specific Azure Active Directory ServiceNow configuration for your organization.
4. We must configure the user properties that are used by Azure Active Directory to provision user information into ServiceNow
5. We must configure the user properties that are user by azure Active directory to enable Single Sign-On for user into ServiceNow.

ServiceNow needs the following user properties to be provided for a user:

| Property | Property in Azure Active Directory | Property in Active Directory |
| --- | --- |
| active (required, "1" = active, "0" = inactive) | softDelete | active |
| email | email | Mail | 
| first_name | firstName | givenName |
| last_name | lastName | sn |
| phone | telephoneNumber | telephoneNumber |
| user_name (required) | userPrincipalName | userPrincipalName |
| title | jobTitle | title |
| city | city | l |
| mobile_phone | mobilePhone | MobilePhone |
| state | state | st |
| street | street | StreetAddress |
| zip | zipCode | postalCode |
 
As you noticed, there are only two properties ("user_name" and "active") that are required for users to sign in into ServiceNow - but for ServiceNow to function properly you will probably want to provision many of the other properties as well.

#### Active Directory

As a first step we need to verify that the data we need to provision into ServiceNow is actually available in the on premises Active Directory. To do that you can use the Attribute Editor: in Active Directory Users and Computers, select View -> Advanced Features, then open a user object and click on the "Attribute Editor" tab. You will now see a list of all user properties that exist in your Active Directory, and you can see (or edit) the values of these properties for the user you selected. When we scroll down the list of properties you will notice that while you can find all of the data we want to provision into ServiceNow, most of the property names in Active directory are different from what is needed in ServiceNow. Fortunately we can configure Azure Active Directory to map the properties with the correct property names in ServiceNow, we'll look at that later.

We will use the following Active Directory properties:

+ userPrincipalName
+ sn
+ mail
+ givenName
+ accountEnabled
+ l (city)
+ sn (surname)
+ st (state)
+ streetAddress
+ postalCode
+ telephoneNumber
+ title

+ userPrincipalName
+ isSoftDeleted
+ mail
+ givenName
+ surname
+ telephoneNumber
+ jobTitle
+ city
+ mobile
+ state
+ streetAddress
+ postalCode

#### Azure AD Connect
If you have not yet installed Azure Active Directory Connect, you can do so by following [these steps](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect#install-azure-ad-connect). 
We will need to configure Azure Active Directory Connect to ensure that the property values we need for ServiceNow are propagated to the Azure Active Directory. We need to use the Configuration Wizard in Azure Active Directory connect to do this. If you open the wizard and authenticate you can forward to the page that allows you to configure "Directory Extensions". On this page, you must verify that 


#### Azure Active Directory - ServiceNow as a SaaS app

##### User Provisioning settings

##### Single Signon Settings