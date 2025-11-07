# Salesforce Barcode Scanner Setup Guide

![Screenshot](images/SalesforceBarcodeScannerscreenshot.jpg)

A comprehensive guide to implementing barcode scanning functionality within your Salesforce environment.

> **Note:** This repository contains instructions only. You do not need to clone this repository to implement the functionality. Simply follow the steps outlined below.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [1. Create a Formula Field for the QR Code](#1-create-a-formula-field-for-the-qr-code)
  - [2. Install the Unofficial Salesforce Barcode Scanning Package](#2-install-the-unofficial-salesforce-barcode-scanning-package)
  - [3. Create the Screen Flow](#3-create-the-screen-flow)
  - [4. Create the Lightning App for the Screen Flow](#4-create-the-lightning-app-for-the-screen-flow)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)

---

## Overview

This guide provides step-by-step instructions to build and configure a barcode scanning feature in Salesforce. This functionality allows users to leverage their mobile device's camera to scan barcodes through the Salesforce mobile app. This can then be used to query for records like Assets, Products, or custom objects within your Salesforce org. This is ideal for use cases such as field service asset tracking, inventory management, and many more use cases.

---

## Prerequisites

Before you begin, ensure you have the following:

>**1: QR Code API Link**
>
>```
> https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=
> ```

>**2: Salesforce Setup Access**
>
>- Ability to create custom fields
>- Ability to create flows
>- Ability to create lightning apps

>**3: Salesforce Mobile App**
>
>- Needed to test and use the functionality
>- MUST be set to "Mobile Only" mode when testing and using

>**4: Unofficial Salesforce Barcode Scanning Package**
>
>- [Production Org Package](https://login.salesforce.com/packaging/installPackage.apexp?p0=04t5G000004fz7hQAA)
>- [Sandbox Org Package](https://test.salesforce.com/packaging/installPackage.apexp?p0=04t5G000004fz7hQAA)
>- [Article](https://unofficialsf.com/from-josh-dayment-barcode-scanning-component-for-flow/) with details on the package

---

## Setup Instructions

Follow these steps to build the barcode scanner functionality.

### 1. Create a Formula Field for the QR Code

1. Go to the ***Object Manager*** in Setup, select the Object you want to add the QR Code field to (the rest of the instructions will assume you chose the *Asset* object).
2. Open the ***Fields & Relationships*** section, then click the ***new*** button.
3. Select the ***Formula*** data type then click ***next***
4. Add a ***Field Label*** for the field (ex. *QR Code*) then click `tab` to autofill the ***Field Name***
5. Select the ***Text*** Formula Return Type then click ***next***
6. Add the following code to the formula box: `IMAGE("https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=" + Id, "QR Code")`

 > **NOTE:**
 >This code creates a QR Code for the record ID of the Asset, if you want to use a different field value for the content of the QR Code, use the ***Insert Field*** tool to add the name of the desired field in replacement of the `Id` portion of the formula above. For example, if you wanted to include the ID of the Account related to the Asset instead of the Asset ID, the formula would look like this: `IMAGE("https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=" + Account.Id, "QR Code")`

7. Once you have added the formula content, click the ***Next*** button.
8. Add the desired Field-Level Security settings, then click the ***Next*** button.
9. Select the Page Layouts to add the new field to, then click the ***Save*** button.

> **OPTIONAL:**
>Feel free to open the Page Layout and modify where the field is placed to verify the QR Code shows up correctly.

### 2. Install the Unofficial Salesforce Barcode Scanning Package

1. Access one of the following links and login to your Salesforce org to view the package installation page. If you are configuring a Production Org then use this **[link](https://login.salesforce.com/packaging/installPackage.apexp?p0=04t5G000004fz7hQAA)**, if you are configuring in a Sandbox Org then use this **[link](https://test.salesforce.com/packaging/installPackage.apexp?p0=04t5G000004fz7hQAA)**
2. Based on your preference, either choose the ***Install for Admins Only*** (if you are just looking to test), or select the ***Install for All Users*** option (if you are finished testing). Then click the ***Upgrade*** button to commend the package installation.

### 3. Create the Screen Flow

This process involves several stages: setting up the scanner, fetching the records, defining user actions, processing updates, and confirming the changes.

#### A. Initial Flow and Scanner Screen Setup

First, we'll create the flow and design the initial screen that holds the barcode scanner component.

1. Access Setup, search for and access the ***Flows*** tab.
2. Click the ***New Flow*** button, then select the ***Screen Flow*** flow type option.
3. Below the ***Screen Flow*** start element, click the ***+*** button to add a new element. Select the ***Screen*** option.
4. Fill in the ***Label*** screen property on the right to give the new screen element a name, then click the `Tab` button to autofill the ***API Name***.
5. From the ***Components*** section on the left side of the screen, scroll down until you find the ***barcodeScanner*** component under the *Custom* section. Drag the component into the middle of the screen element to add it.
6. Fill in the ***API Name*** field on the right side of the screen for the barcodeScanner component you just added to give it a name.
7. Feel free to leave the ***Auto Navigation*** property blank (unless you plan to only scan a single record at a time, then setting this to `{!$GlobalConstant.True}` may streamline things).
8. I suggest leaving the ***Button Icon*** and ***Button Label*** properties *as is*, but feel free to make adjustments.
9. If you plan on performing bulk scans on more than one record at a time, set the ***Continuous Scan*** property to `{!$GlobalConstant.True}`.
10. Fill in the ***Label*** property with a fitting name for the component (ex. "Barcode Scanner").

#### B. Fetching Records Based on Scans

Next, we'll add a step to get the Salesforce records that correspond to the barcodes you scanned.

1. Below the screen element you just created, click the ***+*** button to add a new element. Select the ***Get Records*** element.
2. Fill in the ***Label*** for the get records element, then click the `Tab` button to autofill the ***API Name***.
3. Search and select the ***Object*** that you created the QR Code field for.
4. For the condition, set the ***Field*** to the field you used in the QR Code formula (ex. *Asset ID*, which is what was used by adding the `Id` portion of the formula to the image URL).
5. If you are using the barcode scanning for continuous (bulk scans) then set the *Operator* to `In`. If you plan to only scan a single record at a time, set the *Operator* to `Equals`.
6. If you are using the barcode scanning for continuous scans, then for the *Value* field click *the screen you just created*, then the *Screen Component* (ex. "Barcode Scanner"), then finally `allscannedBarcodes`. If you plan to only scan a single record at a time, then for the *Value* field click *the screen you just created*, then the *Screen Component* (ex. "Barcode Scanner"), then finally `scannedBarcode`.
7. If you are using continuous scans, set the *How Many Records to Store* value to ***All records***. If you are using the single scan, set the value to ***Only the first record***.

#### C. Designing the User Action Screen

This screen will display the scanned records in a table and allow the user to select an action to perform on them.

1. Below the get records element you just created, click the ***+*** button to add a new element. Select the ***Screen*** element.
2. This step is where you have the flexibility to decide what kind of automated actions you want to apply to the scanned record(s). You may choose to add components like a checkbox, choice lookup, date, or other type of field to this screen (which could trigger all kinds of things, even other subflows!). For this example, select the checkbox component on the right side of the screen and drag it to the top of the screen element canvas. Give the checkbox component a ***Label*** and ***API Name*** (ex. "Installed").
3. In the components section on the left side of the screen, search and drag the ***Data Table*** component onto the screen element canvas below the checkbox. Fill in the ***Label*** and ***API Name*** for the data table component.
4. Fill in the ***Source Collection*** field with the record collection variable that was automatically created for the record(s) in the previous get records element. Feel free to check the ***Show Search Bar*** checkbox, or just leave it unchecked if you don't need the ability to search through the scanned record(s).
5. In the *Configure Rows* section, I recommend selecting the ***Multiple*** option, then set the "Minimum Selection" value to ***1***. Set the "Default Selection" to the ***same record collection variable*** as in step 4.
6. Configure the columns for the field values you wish to see displayed in the data table component after scanning. I recommend including things like the record name, the ID, and whatever field you plan on making changes to with the automation (ex. the Status field for Asset records). Once you are satisfied with the screen element's properties, click the ***Done*** button.

#### D. Processing Record Updates

Now, we'll build the logic to loop through the selected records and update them based on the user's input.

1. I won't dive into details here as this could vary depending on the automations you wish to make, but in this step you should add a Loop element to go through each of the record(s) you selected to apply the desired changes. Add two Assignment elements inside of the Loop (one to set the new value for each record, and another to store the record in a new collection to be used later in the flow).
2. Once you have finished creating the Loop element and the Assignment elements inside of it (ex. If the "Installed" checkbox is checked, change the Status for each of the scanned Assets to "Installed"), your next step is to add a ***Update Records*** element directly below the Loop (you MUST ensure the Update Records element is OUTSIDE of the loop).
3. Fill in the ***Label*** for the Update Records element and click `Tab` to autofill the ***API Name***.
4. Select the ***Use the IDs and all field values from a record or record collection*** option, then search and select the record collection variable you created from the Assignment element to store the updated records inside of the Loop element.

#### E. Creating the Confirmation Screen

This final screen will show the user which records have been successfully updated.

1. Below the update records element, add a new ***Screen*** element (this will be the final screen of the flow). Fill in the ***Label*** and then click `Tab` to autofill the ***API Name***.
2. On the right side of the screen, find a drag the ***Data Table*** component onto the screen element canvas. Give the data table an ***API Name*** and ***Label***.
3. Set the "Source Collection" field value to the *record collection variable you created to store the updated records* with the Assignment element. In the "Configure Rows" section, ensure the ***View Only*** option is selected.
4. Finally, fill out the "Configure Columns" section (I recommend adding the same columns you had on the previous data table). Once finished, click the ***Done*** button.

#### F. Finalizing the Flow

The last step is to save and activate your new screen flow.

1. Once all these steps are complete, you can ***Save***, give the flow a name, and ***Activate*** the screen flow.

### 4. Create the Lightning App for the Screen Flow

1. Go to Setup, search for and select ***Lightning App Builder***. Then click the ***New*** button.
2. Selec the ***Ape Page*** option, then click the ***Next*** button.
3. Fill in the ***Label*** with a relevant name for the app (ex. "Scanner").
4. For the form factor, select the ***One Region*** option, then click the ***Done*** button.
5. Once on the *Lightning App Builder* page, on the left side of the screen locate the ***Flow*** component and drag it to the top and center of the app page.
6. On the right side of the screen, in the *Flow* field, search and select the *screen flow you justed created*. Click the ***Save*** button, then ***Activate*** button on the popup.
7. Feel free the change the icon for the Lightning App and select either the *Activate for all users* or *Activate for system administrators only* option in the Page Activation section of the "Page Settings" tab.
8. Click on the ***Mobile Navigation*** tab, then click the ***Add page to app*** button. Feel free to move the app up or down in the mobile navigation menu to fit your liking. Then finish by clicking the ***Save*** button.

> **NOTE:**
>This is the end of the configuration, great work! You are now ready to test.

---

## Usage

While the assumption is that the QR Codes would be printed out and attached to items for labeling and future scanning, this usage guide will explain the steps to testing scanning QR Codes for a record open on Desktop via the Salesforce mobile app.

1. On you computer, open a record for the Object you just added the QR Code formula field to. You should see a QR Code image displayed on the record.
2. Open the Salesforce mobile app on your phone or tablet. Click the ***Menu*** button in the bottom bar of the app screen, then click the ***Mobile Only*** setting (you may have to scroll either up or down to find it).
3. Open the ***Scanner*** Lightning App that you previously created (or whatever else you named the app).
4. Click the "Scan" button (unless you named it something else), once the camera has opened, scan the QR Code for the record on your computer screen with your phone or tablet. You will see a green check icon to confirm the scan was successful. If you wish to scan just the single record, you can click the ***x*** icon to close the scanner and process the record. If you want to perform a bulk scan, open an additional records and continuing scanning until all the desired records have been scanned.
5. Review the scanned records in the data table before pressing the ***Next*** button to continue to processing.
6. Select what action you want to perform to the scanned record(s) to complete the process (ex. check the "Install" checkbox)!

> **NOTE:**
>This all comes down to what processes you chose to automate. For example, a use case might be if the "Install" checkbox is selected for Asset records, then the loop updates the Status picklist value of each Asset to `Install`, and set the Installation Date field to the flows runtime date.

Using the foundation of the QR Code generation and Barcode Scanning package, Salesforce flows can extend these tools to a fit a variety of different use cases.

---

## Troubleshooting

- **Issue:** Barcode scanner does not open.
  - **Solution:** Ensure you are using the Salesforce mobile app on a compatible device, the Salesforce mobile app MUST be in the "Mobile Only" mode (this can be selected in the Menu). The scanner functionality is not available on desktop browsers (but you can display QR Codes there).
- **Issue:** No records are found after scanning.
  - **Solution:** Verify that the scanned barcode exists on a record in Salesforce and that the user has permission to view that record and the field they are trying to scan.
