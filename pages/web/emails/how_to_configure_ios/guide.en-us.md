---
title: Configuring an email address on iPhone and iPad
slug: email_hosting_iphone_ios_91_configuration
excerpt: Find out how to configure an email address on iPhone and iPad
section: Configure on smartphone
order: 1
---

**Last updated 5th May 2020**

## Objective

You can configure email addresses on email clients and applications, if they are compatible. By doing so, you can use your email address through your preferred device.

**Find out how to configure an email address on iPhone and iPad.**

## Requirements

- You need to have an MX Plan email address, available as part of our [Web Hosting plans](https://www.ovh.com/world/web-hosting/)
- You need to have the Mail app installed on your device.
- You need to have the required credentials for the email address you would like to configure.

> [!primary]
>
> This guide can be used for the following iOS versions: iOS 7 or later.
>

## Instructions

There are two ways of adding your email address to the Mail app:

- **In just a few taps, via our Apple Devices tool:** Tap the following link: <https://autodiscover.mail.ovh.net/AppleDevices/> and follow the configuration steps.

- **You can also follow your device’s configuration assistant**.

From this point onwards, this guide will only cover email configuration using your device.

### Step 1: Add the account

On your device’s homepage, go to `Settings`{.action}. There are two ways of adding an account, depending on the iOS version you are using:

- **For iOS 7, 8, 9 and 10:** Go to `Mail, Contacts, Calendar`{.action}, then `Add account`{.action}. Choose `Other`{.action}, then `Add Mail Account`{.action}.

- **For iOS 11:** Go to `Accounts and passwords`{.action}, then `Add account`{.action}. Choose `Other`{.action}, then `Add Mail Account`{.action}.

![Exchange](images/configuration-mail-ios-step1.png){.thumbnail}

Enter your account information:

|Information|Description|
|---|---|
|Name|Enter the sender name that you wish to be displayed when sending emails from this address.|
|Email address|Enter your full email address.|
|Password|Enter the password for your email address.|
|Description|Enter a name that will distinguish this account from any other accounts added in your Mail app.|

Tap `Next`{.action}, and enter the information requested:

|Information|Description| 
|---|---| 
|IMAP or POP|We recommend using **IMAP** (selected by default). You can also select **POP** (emails stored locally on your Mail app) in the drop-down menu.|
|Host name (incoming)|Enter the server “imap.mail.ovh.ca”.|
|Username (incoming)|Enter your full email address.|
|Password (incoming)|Enter the password for your email address.|  
|Host name (outgoing)|Enter the server “smtp.mail.ovh.ca”.|
|Username (outgoing)|Enter your full email address.|
|Password (outgoing)|Enter the password for your email address.| 

Then tap `Next`{.action}. If all the information you have entered is correct, you will be able to log in to your account straight away.

![Exchange](images/configuration-mail-ios-step2.png){.thumbnail}

When you are asked to select the apps you want to use with your account, ensure that `Mail`{.action} is ticked, so that the application works correctly with your email address, then tap `Save`{.action}.

To check that the account has been correctly configured, you can send a test email using the Mail app on your device.

If you need to enter any settings manually in your account preferences, the technical settings you will need to use are listed below:

- **for IMAP configuration**

|Server type|Server name|SSL|Port|
|---|---|---|---|
|Incoming|imap.mail.ovh.ca|Yes|993|
|Outgoing|smtp.mail.ovh.ca|Yes|465|

- **for POP configuration**

|Server type|Server name|SSL|Port|
|---|---|---|---|
|Incoming|pop.mail.ovh.ca|Yes|995|
|Outgoing|smtp.mail.ovh.ca|Yes|465|

### Step 2: Use the email address

Once you have configured your email address, you can start using it! You can now send and receive emails.

OVHcloud also has a web application you can use to access your email account from your browser. You can access this application via the following address: <https://www.ovh.com/world/mail/>. You can log in using your email credentials.

## Go further



Join our community of users on <https://community.ovh.com/en/>.
