---
layout: post
comments: true
title: "Connecting Juniper Sky Enterprise to Aerohive HiveManager"
description: "Sky Enterprise can monitor your Aerohive APs, but documentation on how to do so is a bit hard to find. This post will explain how to link Sky Enterprise to HiveManager so that you can monitor your Aerohive APs from within Sky Enterprise."
#thumb_image: "documentation/sample-image.jpg"
tags: [juniper, aerohive, guides]
---
## Introduction
Juniper Sky Enterprise is a neat little application that can manage and provision SRX, EX, and NFX series products from Juniper Networks. Sky Enterprise also allows you to monitor (but not configure) any Aerohive APs that you have deployed, however the documentation on how to do so is [hidden away on the OneConfig website.](https://oneconfig.com/support/hivemanager-api-integration/)

This post will walk through the process of adding your Aerohive APs to your Sky Enterprise inventory.

In short:
- To link Sky Enterprise and HiveManager, we need to give Sky Enterprise an `Owner ID` and `API Authorization Token` from HiveManager. This allows Sky Enterprise to utilize HiveManager's API to pull information about your APs.
- You can generate API tokens from within HiveManager, but you require a `Client ID` in order to do so.
- Normally to obtain a `Client ID`, we would register and create an application at the Aerohive Developer site, however in this case, Sky Enterprise is the application itself that we want to integrate with, so we will use their Client ID.

Let's get started!

---

## Step 1 - Retrieve your Owner ID from HiveManager
[Login to Aerohive HiveManager](https://cloud.aerohive.com/login) using your existing credentials.

Hover over your name in the top right corner and click __About HiveManager__

{% include image.html path="2018-07-24/1.png"
                      path-detail="2018-07-24/1.png"
                      alt="Click 'About HiveManager' to obtain your Owner ID" %}

Your __Owner ID__ is also known as your the `VHM Id`.

{% include image.html path="2018-07-24/2.png"
                      path-detail="2018-07-24/2.png"
                      alt="Your Owner ID is the VHM Id" %}

__Copy down the `VHM Id` displayed. You will need this in a later step.__

---

## Step 2 - Generate an API Token in HiveManager
Hover over your name in the top right corner again, but this time click __Global Settings__

{% include image.html path="2018-07-24/3.png"
                      path-detail="2018-07-24/3.png"
                      alt="Access the Global Settings for HiveManager" %}

Click __API Token Management__ under __API__ in the menu on the left. Then click the `+` button to generate a new token.

In the popup, paste in the code `e4ba0053`.

{% include image.html path="2018-07-24/4.png"
                      path-detail="2018-07-24/4.png"
                      alt="Generating the API Access Token needed in Sky Enterprise" %}

>`e4ba0053` is the Client ID for Sky Enterprise. It is essential that you use this code or else you will run into issues with your credentials later.

Click __Generate__ to create an API Access Token.

__Copy down the `Access Token` displayed. You will need this in a later step.__

---

## Step 3 - Get the URL of your HiveManager Region
While browsing HiveManager, note down the URL in use. Copy from the `https://` part through to the `.com`

{% include image.html path="2018-07-24/5.png"
                      path-detail="2018-07-24/5.png"
                      alt="Copy the highlighted part of the HiveManager URL" %}

For example: For the Australian Data Center, the URL to be copied will be:

{% highlight text %}
https://cloud-aus.aerohive.com
{% endhighlight %}

---

## Step 4 - Linking Sky Enterprise to HiveManager
[Login to Juniper Sky Enterprise](https://skyenterprise.juniper.net/) using your existing credentials.

Click __Settings__ in the menu bar at the top. Scroll down to the __Aerohive__ option and tick the checkbox.

{% include image.html path="2018-07-24/6.png"
                      path-detail="2018-07-24/6.png"
                      alt="Enabling Aerohive monitoring in Sky Enterprise" %}

- Copy your `Owner ID` (AKA `VHM Id`) obtained in Step 1 into the __Owner ID__ field.
- Copy the `Access Token` generated in Step 2 into the __Authentication Token__ field.
- Copy the HiveManager URL from Step 3 into the __Regional Data Center__ field.

When you are done, scroll to the bottom of the page and click __Update__. A green notification from Sky Enterprise will let you know all is well.

If Sky Enterprise responds with an "invalid credentials" message, check that they information you have entered into the fields above is correct.


---

## Step 5 - Verify your Aerohive APs can be viewed in SKy Enterprise
If the above step was successful, you should now have a __Wifi APs__ tab under the __Devices__ menu. This will list all of the APs you have in your HiveManager inventory.

{% include image.html path="2018-07-24/7.png"
                      path-detail="2018-07-24/7.png"
                      alt="List of APs in Sky Enterprise" %}

You can click the dropdown box/link next to each AP for more information, or a link back to HiveManager.

{% include image.html path="2018-07-24/8.png"
                      path-detail="2018-07-24/8.png"
                      alt="Viewing AP information in Sky Enterprise" %}
