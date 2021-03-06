---
title: Windows C-Sharp SDK
---
## Overview

### SDK Details

Current version: **0.1.0**

Branch links consist of an app-specific base domain (*testbed-windows.app.link*, for example) followed by an alphanumeric suffix. This suffix, referred to as the *alias*, can be either randomly generated (*FCIWxFeBGx*, for example) or manually-defined (*testlink*, for example). The app-specific base domain used to create the links is termed the *Branch link domain*.

Note that when using the 'app.link' default link domain there is a second, alternate, form of the link domain that comes into play when displaying Branch Deepviews - the *alternate link domain*. This *alternate link domain* sees the domain's '.app.link' string replaced by '-alternate.app.link'. If you are using a default app.link link domain, both the regular and alternate domains will be needed when configuring a project to support deep linking.

!!! info "Working Example"
	Please refer to our [TestBed-Windows](https://github.com/BranchMetrics/windows-branch-deep-linking) project for a working example the uses Branch.

A Branch-integrated app called TestBed-Windows and assigned the default Branch link domain *testbed-windows.app.link* has the following Branch links:

- https://testbed-windows.app.link/FCIWxFeBGx
- https://testbed-windows.app.link/testlink

## Integrate Branch

### Configure Branch

Complete the **Basic integration** within [Configure your dashboard](/dashboard/integrate/)

    ![image](/_assets/img/pages/dashboard/fallback.png)

The Branch key can be found on the [General Settings](https://dashboard.branch.io/settings) page. The app's Branch link domain  will be listed at the very bottom of the [Link Settings](https://dashboard.branch.io/link) page.

### Adding the Branch SDK to a Windows Solution

#### **Adding the Json Libary**

First need add Newtonsoft.Json from NuGet

 1. Right-click on project in solution explorer and click on `Manage NuGet Packages...`
 2. Click on `Browse` tab, find `Newtonsoft.Json`and install it

#### **Adding the Branch SDK**

1. Clone this repository to the local machine: `https://github.com/BranchMetrics/windows-branch-deep-linking.git`
2. Add the `branch_debug_0.1.0` or `branch_0.1.0` libary to project from DLLs folder

___

### Integrating the Branch SDK

**I. Create project in visual studio**

* Open Visual Studio and create a new Blank App (Windows Universal) project

**II. Register app to  handle deep link**

* Select `Package.appxmanifest` in Solution Explorer and press `F7`

* Add the <notranslate>**Windows.appUriHandler**</notranslate> extension with your link:

```xml
		<Applications>
			<Application>
				<Extensions>
					<uap3:Extension Category="windows.appUriHandler">
						<uap3:AppUriHandler>
							<uap3:Host Name="testbed-windows.app.link" />
						</uap3:AppUriHandler>
					</uap3:Extension>
				</Extensions>
			</Application>
		</Applications>
```

* Add `DeepLinkHandler.cs` in your project and write in it notification system that the application was activated by deep link for MainPage.xaml.cs, for example:

```csharp
        public static class DeepLinkHandler {
            public static event Action<string> OnAppDeepLinkEvent = delegate { };

            public static void OnAppDeepLink(string url) {
                OnAppDeepLinkEvent.Invoke(url);
            }
        }
```

* Navigate to <notranslate>**App.xaml.cs**</notranslate> in your app Visual Studio solution and in <notranslate>**OnActivated()**</notranslate> add handling for linked content:

```csharp
		protected override void OnActivated(IActivatedEventArgs e) {
			Frame rootFrame = Window.Current.Content as Frame;

			if (rootFrame == null) {
			// Create a Frame to act as the navigation context and navigate to the first page
			rootFrame = new Frame();
			rootFrame.NavigationFailed += OnNavigationFailed;

			Window.Current.Content = rootFrame;
			}

			Type deepLinkPageType = typeof(MainPage);
			if (e.Kind == ActivationKind.Protocol) {
			var protocolArgs = (ProtocolActivatedEventArgs)e;
			DeepLinkHandler.OnAppDeepLink(protocolArgs.Uri.AbsoluteUri);
			}

			if (rootFrame.Content == null) {
			// Default navigation
			rootFrame.Navigate(deepLinkPageType, e);
			}

			// Ensure the current window is active
			Window.Current.Activate();
		}
```
* Subscribe to `OnAppDeepLink` event in MainPage.xaml.cs and in callback handler call InitSession with uri
* Change `OnSuspending` method in App.xaml.cs

```csharp
		private async void OnSuspending(object sender, SuspendingEventArgs e)
		{
			var deferral = e.SuspendingOperation.GetDeferral();
			//TODO: Save application state and stop any background activity
			await LibraryAdapter.GetPrefHelper().SaveAll();
			deferral.Complete();
		}
```

**III. Configuring**

* In `Solution Explorer` in folder `Assets` create folder `Configs` and in it folder create two text files (.txt) with names `live_branch_key.txt` and `test_branch_key.txt` and fill they with branch keys from your app dashboard
* You may configure your app with that methods:
	* `Branch.I.SetDebug(bool isDebug);` - If `isDebug` is true then app will use `Test` Branch key if specified. Otherwise, the app will use the `Live` Branch key.
	* `Branch.I.SetNetworkTimeout(int timeout);` - That method is set timeout for all requests
	* `Branch.I.SetMaxRetries(int maxRetries);` - That method is set max retries for failed requests
	* `Branch.I.SetRetryInterval(int retryInterval);` - That method is set retry interval between request retries
	* `Branch.I.SetRequestMetadata(string key, string value)` - That method is set common metadata for all requests
	* `Branch.I.DisableTracking(bool disableTracking);` - This method determines whether it is possible to use user data **(Many functions will not be available if tracking is disabled)**

### Example code

To help you get started, we have provided this example for learning how to to write Branch UWP projects - [UWP Example](https://bitbucket.org/antonargunov/branch-windows-sdk/src/master/Testbed-Windows-UWP/)

## Configuring a Win32 App

### Adding the Branch SDK to a Windows Solution

#### **Adding the Branch SDK**

1. Clone this repository to the local machine: `https://github.com/BranchMetrics/branch-windows-sdk.git`
2. Add the `branch_win32_debug_0.1.0` or `branch_win32_0.1.0` libary to project from DLLs folder

### Integrating the Branch SDK

**I. Create project in visual studio**

* Open Visual Studio and create a new Console App (.NET Framework) project

**II. Configuring**

* In <notranslate>**Output build**</notranslate> create folder `Configs` and in it folder create two text files (.txt) with names `live_branch_key.txt` and `test_branch_key.txt` and fill they with branch keys from your app dashboard
* You may configure your app with that methods:
	* `Branch.I.SetDebug(bool isDebug);` - If `isDebug` is true then app will use `Test` Branch key if specified. Otherwise, the app will use the `Live` Branch key.
	* `Branch.I.SetNetworkTimeout(int timeout);` - That method is set timeout for all requests
	* `Branch.I.SetMaxRetries(int maxRetries);` - That method is set max retries for failed requests
	* `Branch.I.SetRetryInterval(int retryInterval);` - That method is set retry interval between request retries
	* `Branch.I.SetRequestMetadata(string key, string value)` - That method is set common metadata for all requests
	* `Branch.I.DisableTracking(bool disableTracking);` - This method determines whether it is possible to use user data **(Many functions will not be available if tracking is disabled)**

### Example code

To help you get started, we have provided this example for learning how to to write Branch UWP projects - [Win32 Example](https://bitbucket.org/antonargunov/branch-windows-sdk/src/master/Testbed-Windows-Win32/)

## Implement Branch Features

### Initialize Branch

```csharp
Branch.I.InitSession(new BranchInitCallbackWrapper((buo, linkProperties, error) => { }));
Branch.I.InitSession(new BranchInitCallbackWrapper((parameters, error) => { }));
```

### Retrieve install (install only) parameters

If you ever want to access the original session params (the parameters passed in for the first install event only), you can use this line. This is useful if you only want to reward users who newly installed the app from a referral link or something.

```csharp
var installParams = Branch.I.GetFirstParams();
```

### Persistent identities

Often, you might have the own user IDs, or want referral and event data to persist across platforms or uninstall/reinstall. It's helpful if you know the users access the service from different devices. This where we introduce the concept of an 'identity'.

To identify a user, just call:

```csharp
Branch.I.SetIdentity("User1488", (referringParams, error) => { //callback });
```

### Logout

If you provide a logout function in the app, be sure to clear the user when the logout completes. This will ensure that all the stored parameters get cleared and all events are properly attributed to the right identity.

**Warning** this call will clear the referral credits and attribution on the device.

```csharp
Branch.I.Logout((logout, error) => { //callback });
```

### Tracking User Actions and Events

Use BranchEvent class to track special user actions or application specific events beyond app installs, opens, and sharing. You can track events such as when a user adds an item to an on-line shopping cart, or searches for a keyword etc. BranchEvent provides an interface to add content(s) represented by a BranchUniversalObject in order to associate content(s) with events. You can view analytics for the BranchEvents you fire on the Branch dashboard.  *BranchEventType* enumerate the most commonly tracked events and event parameters that can be used with BranchEvent for the best results. You can always use custom event names and event parameters.

```csharp
BranchEvent ev = new BranchEvent("test_custom_events")
    .SetDescription("Test description")
    .SetTransactionID("322")
    .AddCustomDataProperty("TestProperty", "TestValue");
ev.LogEvent();
```

### Generating Branch links

Branch links can be created in-app as well as in many other ways. When they are, and setIdentity has been called to associate a User ID with the current user session, Branch links will be associated with that User ID.

```csharp
// you can access this data from any instance that installs or opens the app from this link

BranchUniversalObject branchUniversalObject = new BranchUniversalObject()
   .SetCanonicalIdentifier("item/12345")
   .SetCanonicalUrl("https://branch.io/deepviews")
   .SetContentIndexingMode(BranchUniversalObject.ContentIndexModes.PRIVATE)
   .SetLocalIndexMode(BranchUniversalObject.ContentIndexModes.PUBLIC)
   .SetTitle("My Content Title")
   .SetContentDescription("my_product_description1")
   .SetContentImageUrl("https://example.com/mycontent-12345.png")
   .SetContentExpiration(DateTime.UtcNow)
   .SetContentImageUrl("https://test_img_url")
   .AddKeyWord("My_Keyword1")
   .AddKeyWord("My_Keyword2")
   .SetContentMetadata(
        new BranchContentMetadata().AddCustomMetadata("testkey", "testvalue")
   );

BranchLinkProperties linkProperties = new BranchLinkProperties()
         .AddTag("Tag1")
         .SetChannel("Sharing_Channel_name")
         .SetFeature("my_feature_name")
         .AddControlParameter("$android_deeplink_path", "custom/path/*")
         .AddControlParameter("$ios_url", "http://example.com/ios")
         .SetDuration(100);

string url = branchUniversalObject.GetShortURL(linkProperties);
```

**Note**
You can customize the Facebook OG tags of each URL if you want to dynamically share content by using the following _optional keys in the data dictionary_. Please use this [Facebook tool](https://developers.facebook.com/tools/debug/og/object) to debug the OG tags!

| Key | Value
| --- | ---
| `$og_title` | The title you'd like to appear for the link in social media
| `$og_description` | The description you'd like to appear for the link in social media
| `$og_image_url` | The URL for the image you'd like to appear for the link in social media
| `$og_video` | The URL for the video
| `$og_url` | The URL you'd like to appear
| `$og_app_id` | the OG app ID. Optional and rarely used.

Also, you do custom redirection by inserting the following _optional keys in the dictionary_:

| Key | Value
| --- | ---
| `$desktop_url` | Where to send the user on a desktop or laptop. By default it is the Branch-hosted text-me service
| `$android_url` | The replacement URL for the Play Store to send the user if they don't have the app. _Only necessary if you want a mobile web splash_
| `$ios_url` | The replacement URL for the App Store to send the user if they don't have the app. _Only necessary if you want a mobile web splash_
| `$ipad_url` | Same as above but for iPad Store
| `$fire_url` | Same as above but for Amazon Fire Store
| `$blackberry_url` | Same as above but for Blackberry Store
| `$windows_phone_url` | Same as above but for Windows Store

You have the ability to control the direct deep linking of each link by inserting the following _optional keys in the dictionary_:

| Key | Value
| --- | ---
| "`$deeplink_path` | The value of the deep link path that you'd like us to append to the URI. For example, you could specify "$deeplink_path": "radio/station/456" and we'll open the app with the URI "theapp://radio/station/456?link_click_id=branch-identifier". This is primarily for supporting legacy deep linking infrastructure.
| "`$always_deeplink` | true or false. (default is not to deep link first) This key can be specified to have our linking service force try to open the app, even if we're not sure the user has the app installed. If the app is not installed, we fall back to the respective app store or $platform_url key. By default, we only open the app if we've seen a user initiate a session in the app from a Branch link (has been cookied and deep linked by Branch)


### Referral rewards

In a standard referral system, you have 2 parties: the original user and the invitee. Our system is flexible enough to handle rewards for all users. Here are a couple example scenarios:

1) Reward the original user for taking action (eg. inviting, purchasing, etc)

2) Reward the invitee for installing the app from the original user's referral link

3) Reward the original user when the invitee takes action (eg. give the original user credit when their the invitee buys something)

These reward definitions are created on the dashboard, under the 'Reward Rules' section in the 'Referrals' tab on the dashboard.

Warning: For a referral program, you should not use unique awards for custom events and redeem pre-identify call. This can allow users to cheat the system.

#### Check a reward balance

Reward balances change randomly on the backend when certain actions are taken (defined by the rules), so you'll need to make an asynchronous call to retrieve the balance. Here is the syntax:

```csharp
        Branch.I.LoadRewards(async (changed, error) => {
            //callback
        });
```

#### Redeem all or some of the reward balance

We will store how many of the rewards have been deployed so that you don't have to track it on the end. In order to save that you gave the credits to the user, you can call redeem. Redemptions will reduce the balance of outstanding credits permanently.

```csharp
        Branch.I.RedeemRewards(5, async (changed, error) => {
            //callback
        });
```

#### Get credit history

This call will retrieve the entire history of credits and redemptions from the individual user.  It also implements the IBranchRewardsInterface(see above). To use this call, implement like so:

```csharp
        Branch.I.GetCreditHistory(async (response, error) => {
            //callback
        });
```

The response will return an array that has been parsed from the following JSON:
```json
        [
            {
                "transaction": {
                                   "date": "2014-10-14T01:54:40.425Z",
                                   "id": "50388077461373184",
                                   "bucket": "default",
                                   "type": 0,
                                   "amount": 5
                               },
                "referrer": "12345678",
                "referree": null
            },
            {
                "transaction": {
                                   "date": "2014-10-14T01:55:09.474Z",
                                   "id": "50388199301710081",
                                   "bucket": "default",
                                   "type": 2,
                                   "amount": -3
                               },
                "referrer": null,
                "referree": "12345678"
            }
        ]
```


<notranslate>**referrer**</notranslate>
: The id of the referring user for this credit transaction. Returns null if no referrer is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call.

<notranslate>**referree**</notranslate>
: The id of the user who was referred for this credit transaction. Returns null if no referree is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call.

<notranslate>**type**</notranslate>
: This is the type of credit transaction

1. <notranslate>_0_</notranslate> - A reward that was added automatically by the user completing an action or referral
1. <notranslate>_1_</notranslate> - A reward that was added manually
2. <notranslate>_2_</notranslate> - A redemption of credits that occurred through our API or SDKs
3. <notranslate>_3_</notranslate> - This is a very unique case where we will subtract credits automatically when we detect fraud


#### Link sharing
You can share your generated link with that method:
```csharp
BranchUniversalObject branchUniversalObject = new BranchUniversalObject()
                   .SetCanonicalIdentifier("item/12345")
                   .SetCanonicalUrl("https://branch.io/deepviews")
                   .SetContentIndexingMode(BranchUniversalObject.ContentIndexModes.PRIVATE)
                   .SetLocalIndexMode(BranchUniversalObject.ContentIndexModes.PUBLIC)
                   .SetTitle("My Content Title")
                   .SetContentDescription("my_product_description1")
                   .SetContentImageUrl("https://example.com/mycontent-12345.png")
                   .SetContentExpiration(DateTime.UtcNow)
                   .SetContentImageUrl("https://test_img_url")
                   .AddKeyWord("My_Keyword1")
                   .AddKeyWord("My_Keyword2")
                   .SetContentMetadata(
                        new BranchContentMetadata().AddCustomMetadata("testkey", "testvalue")
                   );

            BranchLinkProperties linkProperties = new BranchLinkProperties()
                     .AddTag("Tag1")
                     .SetChannel("Sharing_Channel_name")
                     .SetFeature("my_feature_name")
                     .AddControlParameter("$android_deeplink_path", "custom/path/*")
                     .AddControlParameter("$ios_url", "http://example.com/ios")
                     .SetDuration(100);

            BranchShareSheetStyle style = new BranchShareSheetStyle("Test share title", "Test share message body");
            style.SetDefaultUrl("https://branch.io/");
            branchUniversalObject.ShowShareSheet(DataTransferManager.GetForCurrentView(), Dispatcher, linkProperties, style);
```
```

<notranslate>**BranchShareSheetStyle**</notranslate>
This class is need for customize "Link sharing"
* Its constructor takes two arguments `title` and `message`
* Method `SetDefaultUrl` allows you set default url if url generating was failed
* Method `SetBitmap` set image for your share

#####!!! `DataTransferManager.GetForCurrentView()` and `Dispatcher` should be called from main thread !!!#####


## Branch Metrics Windows COM
1. [**Introduction**](#1---introduction)
  + [Creating project](#creating-project)
  + [Setup COM Branch Library](#setup-com-branch-library)

2. [**Using of COM Library**](#2---using-of-com-library)
  + [Initialize COM](#initialize-com)
  + [COM Branch class](#com-branch-class)
  + [Using of COM Branch objects](#using-of-com-branch-objects)
  + [Using of COM Branch callbacks](#using-of-com-branch-callbacks)
  + [Example full code](#example-full-code)

## 1 - Introduction

### Creating project
1. Start Microsoft Visual Studio
2. On the <notranslate>**File**</notranslate> menu, point to
    <notranslate>**New**</notranslate>, and then click <notranslate>**Project**</notranslate>. The <notranslate>**New Project**</notranslate> dialog box opens.
3. Under <notranslate>**Installed**</notranslate>, click <notranslate>**Visual C++**</notranslate>.
4. Click <notranslate>**Windows Console Application**</notranslate>.


### Setup COM Branch Library

 1. Build solution
 2. Copy COM Library from
**..\BranchWindowsSdk\DLLs\Debug\branch_debug_0.1.0.tlb** or
**..\BranchWindowsSdk\DLLs\Release\branch_0.1.0.tlb** to
**..\Your Project Folder\Your Project Name\Debug** or
**..\Your Project Folder\Your Project Name\Release**
 3. Import library
```cpp
#import "..\Your Project Name\Debug\branch_debug_0.1.0.tlb"
using namespace branch_debug_0.1.0;
```
or
```cpp
#import "..\Your Project Name\Debug\branch_0.1.0.tlb"
using namespace branch_0.1.0;
```
 4. Include ```<thread>```
```cpp
#include <thread>
```
 ___

## Using of COM Library

### Initialize COM
To initialize the use of a COM, paste this code into your main class.
```cpp
int main()
{
	HRESULT hr = CoInitialize(NULL);
	//In next your code will be here
	CoUninitialize();
}
```
### COM Branch class
To create COM Branch class use this:
```cpp
ICOMBranchPtr comBranch(__uuidof(COMBranch));
```
Next call ``GetBranchInstance`` and ``SetNetworkTimeout()`` methods
```cpp
comBranch->GetBranchInstance(false, "your branch key");
comBranch->SetNetworkTimeout(3000);
```
COM Branch Methods:

 - void GetBranchInstance(bool isLive, string key);
 - void EnableTestMode();
 - void SetDebug(bool isDebug);
 - void DisableTracking(bool disableTracking);
 - bool IsTrackingDisabled();
 - void SetNetworkTimeout(int timeout);
 - void SetMaxRetries(maxRetries);
 - void SetRetryInterval(int retryInterval);
 - void SetRequestMetadata(string key, string value);
 - void EnableSimulateInstall();
 - void DisableSimulateInstall();
 - string GetFirstParams();
 - string GetSessionParams();
 - void SetIdentity(string userID, void* callback);
 - void Logout(void* callback);
 - void RedeemRewards(int count);
 - void RedeemRewards(int count, void* callback);
 - void RedeemRewards(string bucket, int count);
 - void RedeemRewards(string bucket, int count, void* callback);
 - void UserCompletedAction(string action, string metadata);
 - void UserCompletedAction(string action);
 - void UserCompletedAction(string action, void* callback);
 - void UserCompletedAction(string action, string metadata, void* callback);
 - void CancelShareLinkDialog();
 - void GetCreditHistory(void* callback);
 - void GetCreditHistory(string afterID, int length, string order, void* callback);
 - void GetCreditHistory(string bucket, string afterID, int length, string order, void* callback);
 - int GetCredits();
 - void InitSession(string linkUrl = "", bool autoInitSession = false);
 - void InitSession(bool isReferrable, string linkUrl = "");
 - void InitSession(void* callback, string linkUrl = "");
 - void InitSession(bool isReferrable, void* callback, string linkUrl = "");

### Using of COM Branch objects
#### Branch Content Metadata
```cpp
ICOMBranchContentMetadataPtr comMetadata(__uuidof(COMBranchContentMetadata));
```
#### Branch Universal Object
```cpp
ICOMBranchUniversalObjectPtr comBUO(__uuidof(COMBranchUniversalObject));
```
#### Branch Link Properties
```cpp
ICOMBranchLinkPropertiesPtr comLinkProperties(__uuidof(COMBranchLinkProperties));
```
#### Example of use objects:
```cpp
ICOMBranchContentMetadataPtr comMetadata(__uuidof(COMBranchContentMetadata));
comMetadata->AddCustomMetadata("testkey", "testvalue");

ICOMBranchUniversalObjectPtr comBUO(__uuidof(COMBranchUniversalObject));
comBUO->CanonicalIdentifier = "item/12345";
comBUO->CanonicalUrl = "";
comBUO->ContentIndexMode = "PRIVATE";
comBUO->LocalIndexMode = "PUBLIC";
comBUO->Title = "My Content Title";
comBUO->ContentDescription = "my_product_description1";
comBUO->ImageUrl = "https://example.com/mycontent-12345.png";
comBUO->AddKeyword("My_Keyword1");
comBUO->AddKeyword("My_Keyword2");

ICOMBranchLinkPropertiesPtr comLinkProperties(__uuidof(COMBranchLinkProperties));
comLinkProperties->AddTag("Tag1");
comLinkProperties->Channel = "Sharing_Channel_name";
comLinkProperties->Feature = "my_feature_name";
comLinkProperties->AddControlParam("$android_deeplink_path", "custom/path/*");
comLinkProperties->AddControlParam("$ios_url", "http://example.com/ios");
comLinkProperties->MatchDuration = 100;
```
### Using of COM Branch callbacks

#### COM Delegates

 - Init session delegate
```cpp
   void (ICOMBranchUniversalObject buo, ICOMBranchLinkProperties link, LPTSTR error);
```
 - Get Reward History delegate
```cpp
   void (string json, LPTSTR error);
```
 - Get Rewards delegate
```cpp
   void (bool changed, LPTSTR error);
```
 - Logout delegate
```cpp
   void (bool logoutSuccess, LPTSTR error);
```
 - Redeem Rewards delegate
```cpp
   void (bool changed, LPTSTR error);
```
 - Identity user delegate
```cpp
   void (string referrinParams, LPTSTR error);
```
 - User Completed Action delegate
```cpp
   void ();
```

#### Using of callback example

```cpp
void __stdcall InitCallback(
	ICOMBranchUniversalObjectPtr comBUO,
	ICOMBranchLinkPropertiesPtr comLinkProperties,
	LPTSTR error) {
	//your code here
}

int main()
{
	HRESULT hr = CoInitialize(NULL);
	ICOMBranchPtr comBranch(__uuidof(COMBranch));
	comBranch->GetBranchInstance(false,"your branch key");
	comBranch->SetNetworkTimeout(3000);
	comBranch->InitSession_3(&InitCallback, "");
	CoUninitialize();
}
```
### Examples
To help you get started, we have provided this examples for learning how to to write COM Projects.
1. [Example of using COM UWP](https://bitbucket.org/antonargunov/branch-windows-sdk/src/master/Testbed-Windows-COM-UWP/)
2. [Example of using COM Win32](https://bitbucket.org/antonargunov/branch-windows-sdk/src/master/Testbed-Windows-COM-Win32/)
