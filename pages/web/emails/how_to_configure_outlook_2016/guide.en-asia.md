---
title: Configuring your email address in Outlook 2016 for Windows
slug: configuration-outlook-2016
excerpt: Find out how to configure your email address in Outlook 2016 for Windows
section: Configure on computer
order: 1
---

**Last updated 5th May 2020**

## Objective

You can configure email addresses on email clients, if they are compatible. By doing so, you can use your email address through your preferred email application.

**Find out how to configure your email address in Outlook 2016 for Windows.**

## Requirements

- You need to have an MX Plan email address, available as part of our [Web Hosting plans](https://www.ovh.com/asia/web-hosting/).
- You must have Microsoft Outlook 2016 installed on your device.
- You need to have the required credentials for the email address you would like to configure.

> [!primary]
>
> Are you using Outlook 2016 for Mac? Refer to our documentation: [Configuring an email address in Outlook 2016 for Mac](../configuration-outlook-2016-mac/).
>

## Instructions

### Step 1: Add the account

Once you have launched the Outlook app on your device, you can add an account in two different ways.

- **When you start the application for the first time:** A setup wizard will appear and prompt you to enter your email address.

- **If you have already added an account:** Click `File`{.action} in the menu bar at the top of your screen, then `Add account`{.action}.

![mxplan](images/configuration-outlook-2016-windows-step1.png){.thumbnail}

Enter your email address, then click `Advanced options`{.action}. Tick the box that appears next to `Configure my account manually`{.action}, then click `Connection`{.action}.

Select either  **IMAP** or **POP** from the list of account types. We recommend using IMAP. If you would like to store your emails locally on your Outlook application, you can select **POP**.

![mxplan](images/configuration-outlook-2016-windows-step2.png){.thumbnail}

Then fill in the information requested.

- **For incoming mail**

|Information|Description|
|---|---|
|Server|Enter the server “imap.mail.ovh.ca”.|
|Port|Enter port “993”.|
|Encryption method|Select “SSL/TLS”.|
|Require authentication|Do not tick the “Require secure password authentication (SPA) on connection” box.|

- **For outgoing mail**

|Information|Description|
|---|---|
|Server|Enter the server “smtp.mail.ovh.ca”.|
|Port|Enter port “465”.|
|Encryption method|Select “SSL/TLS”.|
|Require authentication|Do not tick the “Require secure password authentication (SPA) on connection” box.|

Once you have entered this information, click `Next`{.action} and enter your email **password**. If all the information you have entered is correct, you will be able to log in to your account straight away.

To check that the account has been correctly configured, you can send a test email.

![mxplan](images/configuration-outlook-2016-windows-step3.png){.thumbnail}

If you need to enter any settings manually in your account preferences, the technical settings you will need to use are listed below:

- **For IMAP configuration**

|Server type|Server name|Encryption method|Port|
|---|---|---|---|
|Incoming|imap.mail.ovh.ca|SSL/TLS|993|
|Outgoing|smtp.mail.ovh.ca|SSL/TLS|465|

- **For POP configuration**

|Server type|Server name|Encryption method|Port|
|---|---|---|---|
|Incoming|pop.mail.ovh.ca|SSL/TLS|995|
|Outgoing|smtp.mail.ovh.ca|SSL/TLS|465|

### Step 2: Use the email address

Once you have configured your email address, you can start using it! You can now send and receive emails.

OVHcloud also has a web application you can use to access your email account from your browser. You can access this application at <https://www.ovh.com/asia/mail>. Log in using your email credentials.

## Go further


Join our community of users on <https://community.ovh.com/en/>.
