---
layout: post
comments: true
title: "Configuring Zscaler Private Access (ZPA) to use Azure AD"
description: "Set Azure Active Directory (AAD) as the Identity Provider (IdP) for the Zscaler Private Access (ZPA) service. Configure SAML Single Sign-On (SSO) and SCIM Provisioning."
thumb_image: "2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/banner.png"
tags: [azure, zscaler, lab, notes]
---

For Zscaler Internet Access (ZIA), see this guide.

> Disclaimer: I am an employee of Zscaler. These are my own notes from a ZPA homelab deployment with Azure AD and may be incorrect or against best practice. You should consult the [Zscaler help portal](https://help.zscaler.com) for official documentation.



## Before you begin

You must have setup users and (optionally) groups in Azure AD. Follow this companion post on how to do so if you haven't already. It's free.

From the Azure Home page, in the side menu, click on **Azure Active Directory** to begin.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad1.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad1.png" %}



## Adding Enterprise Applications for SSO

From the directory side menu under **Manage**, click **Enterprise applications**, then **New application**.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad11.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad11.png" %}

Search for the application you want to integrate with. In this case, `Zscaler`

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad12.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad12.png" %}



## Configuring ZPA to use Azure AD

>  Zscaler Private Access (ZPA)

### 1 - Create the ZPA application

Under the enterprise apps page above, select **Zscaler Private Access (ZPA)** and click **Create**.

The application will be created in Azure AD and you will be redirected to the Overview page.



### 2 - Assign Users and Groups to the Zscaler App in Azure AD

In order for users to be able to use the Zscaler service, you must first assign the Zscaler application to them in Azure AD.

In the side navigation menu for Azure AD, click **Users and Groups**, then **Add user**.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad20.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad20.png" %}

If you are using the free Azure AD plan, then you will only be allowed to assign individual users, not groups to the application.

Click **Users** and select all of the user accounts you wish to assign ZPA access to. Then click **Assign** to finish.



### 3 - Configure the IdP in the ZPA Admin Portal

Open a new tab and navigate to your Zscaler admin portal for ZPA:
https://admin.private.zscaler.com

In the side navigation menu, go to **Administration** > **IdP Configuration**

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa1.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa1.png" %}

Click **Add IdP Configuration** at the top-right of the screen.

Under the **IdP Information** section:

1. Enter a name for the IdP configuration.
   ZPA supports multipe IdPs, hence a name is required to differentiate them.
2. Ensure **User** is selected for **Single Sign-On**.
3. Select the domain you wish to associate with the IdP configuration.
   Zscaler support must add any additional domains to your admin portal first before you can use them.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa2.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa2.png" %}

Click **Next** to continue.

The next tab shows info you'll need to copy and add into Azure AD. Note down the **Service Provider Entity ID** and **Service Provider URL** fields.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa3.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa3.png" %}

Click **Pause** and return to the Azure AD portal.





### 4 - Configure SAML SSO in Azure AD

From the side navigation menu in the Azure AD portal, under **Manage**, click **Single sign-on**.

When prompted to select a SSO method, click **SAML**.

Click the pencil/edit icon next to **Section 1 - Basic SAML Configuration**.

* For **Identifier**, enter the **Service Provider Entity ID** copied from the ZPA portal. This will be in the format:
  `https://samlsp.private.zscaler.com/auth/metadata/<uniqueID>`
* For **Reply URL**, enter the **Service Provider URL** copied from the ZPA portal. This will be in the format:
  `https://samlsp.private.zscaler.com/auth/<uniqueID>/sso`
* For **Sign on URL**, enter the following URL replacing the `<company.com>` with your full domain from Azure AD:
  `https://samlsp.private.zscaler.com/auth/v2/login?domain=<company.com>`
* Leave **Relay State** and **Logout URL** blank.

After saving, you will be prompted to test the configuration: **DO NOT test the configuration at this time**.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad21.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/aad21.png" %}

Under **Section 3 - SAML Signing Certificate**, download the **Certificate (Base64)** and **Federation Metadata XML** file. You will need to upload these to the to the ZPA admin portal in the next step.

IMPORTANT: The certificate file MUST be in `.pem` format. If it downloads as a `.cer` file, change the extension to `.pem`



### 5 - Configure SAML SSO & SCIM Provisioning in the ZPA Admin Portal

Return to the **IdP Configuration** page in ZPA admin portal. Under **Actions**, click the **play/resume button** to continue with the configuration.

* For **IdP Metadata File**, upload the **Federation Metadata XML** file you downloaded from Azure AD. This will automatically populate the **Single Sign-On URL** and **IdP Entity ID** fields.
  If for whatever reason these don't populate, they correspond to the **Login URL** and **Azure AD Identifier** URLs respectively from the SAML setup page in AAD.
* For **IdP Certificate**, upload the Certificate (Base64) file in `.pem` format downloaded from Azure AD. If it downloaded as a `.cer` file, you may need to change the file extension to `.pem` first.
* Set the **Status** to **Enabled**.
* Set **ZPA SAML Request** to **Signed**.
* Set **HTTP-Redirect** to **Disabled**.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa4.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa4.png" %}

Under **SCIM**, set **SCIM Sync** to **Enabled**, and **generate a new bearer token**.

Copy the **SCIM Service Provider Endpoint** URL and **Bearer Token** - you will need these in the next step.

Click **Save** when you are finished.



### 6 - Configure SCIM Provisioning in Azure AD

Go back to Azure AD, and in the side navigation menu, click **Provisioning**.

* Set the **Provisioning Mode** to **Automatic**.
* For **Tenant URL** enter the **SCIM Service Provider Endpoint** URL copied from the admin portal.
  `https://scim1.private.zscaler.com/scim/1/<uniqueID>/v2`
* For **Secret Token**, enter the **Bearer Token** copied from the admin portal.

Click **Test Connection** to verify everything works. **Save** the configuration when the test passes.

IMPORTANT: Make sure to set **Provisioning Status** to **On**. You will need to re-save the configuration for this to stay set.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa5.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa5.png" %}



### 7 - Check User Provisioning Status

You may need to wait for the provisioning cycle to complete before the users are able to log in.

To view the status of user provisioning, click **Provisioning** from the side menu (where you enabled automatic provisioning via SCIM above).

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa6.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa6.png" %}



### 8 - Import SAML attributes to verify IdP Configuration

Form the ZPA admin portal, you need to import SAML attributes from Azure AD. This also allows us to test whether user authentication is working correctly.

1. Open the ZPA admin portal: https://admin.private.zscaler.com
2. In the side navigation menu, go to **Administration** > **IdP Configuration**.
3. Expand the IdP configuration you want to import the attributes from.
4. Under **Import SAML Attributes** select one of the domains you associated with the IdP and click **Import**.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa7.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa7.png" %}

If all is working, you will be prompted to sign in using Microsoft SSO, after which you will be redirected back to the ZPA admin portal.

The JSON string at the bottom of the ZPA window shows all of the SAML attributes, whereas the attributes that appear in the table at the top of the screen are a subset **that haven't been imported yet**. If you don't see any SAML attributes in the table, then this means that they have all already been imported.

{% include image.html path="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa8.png"
                      path-detail="2020-01-02-configuring-zscaler-zpa-to-use-azure-ad.assets/zpa8.png" %}

SAML attributes are used when creating policy. You may wish to rename some of the attributes so they are easy to identify. Eg: `http___[...]_display_name` --> Full name, `http___[...]_given_name` --> First Name, etc

Click **Save** to import the discovered SAML attributes.

You will be redirected to the **SAML Attributes** page where you can edit or delete attributes as necessary.



### 9 - Browser testing

You can test the IdP configuration outside of the ZPA admin portal using the following URL. Replace `<company.com>` with your company's domain name as per Azure AD.

IMPORTANT: Test the URL in a Private/Incognito window.

`https://samlsp.private.zscaler.com/auth/v2/login?domain=<company.com>&ssotype=test`



### 10 - User Roles and Group Mapping to ZPA

TODO - Need to upgrade to a premium Azure AD account to test groups.

https://help.zscaler.com/zpa/configuration-guide-microsoft-azure-ad



## Next Steps

If this was the first IdP you configured and your first time setting up ZPA, you will need to raise a support ticket to have ZPA enabled within the Zscaler App mobile portal so that it can be selectively rolled out to users.