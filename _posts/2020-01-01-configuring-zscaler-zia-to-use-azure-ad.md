---
layout: post
comments: true
title: "Configuring Zscaler Internet Access (ZIA) to use Azure AD"
description: "Set Azure Active Directory (AAD) as the Identity Provider (IdP) for the Zscaler Internet Access (ZIA) service. Configure SAML Single Sign-On (SSO) and SCIM Provisioning."
thumb_image: "2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/banner.png"
tags: [azure, zscaler, lab, notes]
---

For Zscaler Private Access (ZPA), see this guide.

> Disclaimer: I am an employee of Zscaler. These are my own notes from a ZIA homelab deployment with Azure AD and may be incorrect or against best practice. You should consult the [Zscaler help portal](https://help.zscaler.com) for official documentation.


## Before you begin

You must have setup users and (optionally) groups in Azure AD. Follow this companion post on how to do so if you haven't already. It's free.

From the Azure Home page, in the side menu, click on **Azure Active Directory** to begin.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad1.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad1.png" %}



## Adding Enterprise Applications for SSO

From the directory side menu under **Manage**, click **Enterprise applications**, then **New application**.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad11.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad11.png" %}

Search for the application you want to integrate with. In this case, `Zscaler`

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad12.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad12.png" %}



## Configuring ZIA to use Azure AD

>  Zscaler Internet Access (ZIA)

### 1 - Create the Zscaler application

Under the enterprise apps page above, select the Zscaler application that corresponds to cloud you connect to the admin portal with. For example:

* `admin.zscalertwo.net` == **Zscaler Two**
* `admin.zscaler.net` == **Zscaler**

When you have found your application, click **Create**.



### 2 - Assign Users and Groups to the Zscaler App in Azure AD

In order for users to be able to use the Zscaler service, you must first assign the Zscaler application to them in Azure AD.

In the side navigation menu for Azure AD, click **Users and Groups**, then **Add user**.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad15.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad15.png" %}

If you are using the free Azure AD plan, then you will only be allowed to assign individual users, not groups to the application.

Click **Users** and select all of the user accounts you wish to assign ZIA access to. Then click **Assign** to finish.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad16.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad16.png" %}

The selected users will now be able to access the Zscaler application (after provisioning and synchronization with Zscaler is configured in the following steps).



### 3 - Configure SAML SSO in Azure AD

From the side navigation menu, under **Manage**, click **Single sign-on**.

When prompted to select a SSO method, click **SAML**.

Click the pencil/edit icon next to **Section 1 - Basic SAML Configuration**.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad13.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad13.png" %}

* For **Identifier**, enter `<zscalercloud>.net`. You would have identified this in Step 1 above.
  For example:
  * For **Zscaler Two**, enter `zscalertwo.net` as the Identifier.
  * For **ZS Cloud**, enter `zscloud.net` as the Identifier.
  * For **Zscaler**, enter `zscaler.net` as the Identifier.
  * For **Zscaler One**, enter `zscalerone.net` as the Identifier.
  * For **Zscaler Three**, enter `zscalerthree.net` as the Identifier.
* For **Reply URL**, enter the following - replacing `<zscalercloud>` with the name of your cloud from above:
  `https://login.<zscalercloud>.net/sfc_sso`
* For **Sign on URL**, enter the same URL as above.
* Leave **Relay State** and **Logout URL** blank.

The screenshot above shows the configuration for the `zscaler.net` cloud.

Next, go to **Section 3 - SAML Signing Certificate**, and download the **Base64** Certificate. You will need to upload this later to the ZIA admin portal.
**IMPORTANT:** The certificate will download as a `.cer` file. Change the file extension to `.pem`, eg: `Zscaler.pem`

Next, scroll down to **Section 5 - Set up Zscaler**, and expand the **Configuration URLs**. Note down the **Login URL** - you will need this later in the ZIA admin portal.



### 4 - Configure SAML SSO & SCIM Provisioning in the Zscaler Admin Portal

Open a new tab and navigate to your Zscaler admin portal for ZIA:
`https://admin.<zscalercloud>.net`

In the side navigation menu, go to **Administration** > **Authentication Settings**

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs1.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs1.png" %}



Select **SAML** as the Authentication Type and click the **Configure SAML** link.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs2.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs2.png" %}

Fill in the details as follows:

* For **SAML Portal URL**, enter the **Login URL** you copied from Azure AD under **Section 5 - Set up Zscaler**.
  `https://login.microsoftonline.com/<uniquetoken>/saml2`
* For **Login Name Attribute** enter: `NameID`
* For **Public SSL Certificate** upload the Base64 certificate you downloaded from Azure AD under **Section 3 - SAML Signing Certificate**.
  It MUST be in `.pem` format. If it downloaded as a `.cer` file, simply change the file extension to `.pem.
* Disable **Sign SAML Request** - Azure AD doesn't support signed SAML responses from Service Providers.
* Disable **Enable SAML Auto-Provisioning** - SCIM is the recommended option for auto-provisioning and should be used where possible instead.
* Check **Enable SCIM-Based Provisioning**. Copy the **Base URL** and **Bearer Token** - you will need both of these in the next section.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs3.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/zs3.png" %}

Save your changes and don't forget to activate!



### 5 - Configure SCIM Provisioning in Azure AD

Go back to Azure AD, and in the side navigation menu, click **Provisioning**.

* Set the **Provisioning Mode** to **Automatic**.
* For **Tenant URL** enter the **Base URL** copied from the admin portal.
  `https://scim.zscaler.net/<uniqueID>/scim`
* For **Secret Token**, enter the **Bearer Token** copied from the admin portal.

Click **Test Connection** to verify everything works. Save the configuration when the test passes.

IMPORTANT: Make sure to set **Provisioning Status** to **On**. You will need to re-save the configuration for this to stay set.

{% include image.html path="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad14.png"
                      path-detail="2020-01-01-configuring-zscaler-zia-to-use-azure-ad.assets/aad14.png" %}



### 6 - Check User Provisioning Status

You may need to wait for the provisioning cycle to complete before the users are able to log in.

To view the status of user provisioning, click **Provisioning** from the side menu (where you enabled automatic provisioning via SCIM above).

If the users look like they have synchronized, then you can see if they are present in the Zscaler admin portal under **Administration** > **User Management**.
