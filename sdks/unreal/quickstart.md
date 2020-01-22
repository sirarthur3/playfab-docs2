---
title: Unreal Engine quickstart
author: v-thopra
description: This guide will help you set up the Unreal Engine, install the PlayFab Marketplace Plugin, and make your first API call in Unreal, using the PlayFab Marketplace Plugin.
ms.author: v-thopra
ms.date: 06/11/2018
ms.topic: article
ms.prod: playfab
keywords: playfab, unreal, playfab marketplace plugin, c++, blueprint
ms.localizationpriority: medium
---

# Unreal Engine quickstart

This quickstart helps you set up Unreal Engine, install the PlayFab Marketplace Plugin, and make your first API call in Unreal, using the PlayFab Marketplace Plugin. You can make your first API call using Blueprints, or C++, or both.

Before you can call any PlayFab API, you must have a [PlayFab developer account](https://developer.playfab.com/en-us/sign-up). 

## Table of contents

- [Unreal project setup](#unreal-project-setup)  
- [Set up your first Blueprint call](#set-up-your-first-blueprint-call)  
  - [Finish and execute with Blueprint](#finish-and-execute-with-blueprint)
- [Set up your first C++ call](#set-up-your-first-c-call)
  - [Finish and Execute with C++](#finish-and-execute-with-c)
- [Deconstruct the Blueprint example](#deconstruct-the-blueprint-example)
- [Deconstruct the C++ code example](#deconstruct-the-c-code-example)
- [Upgrading to the Unreal Marketplace plugin](#upgrading-to-the-unreal-marketplace-plugin)

## Unreal project setup

OS: This guide is written for Windows 10, however, steps should be similar for Macintosh. This guide is created for using Visual Studio 2017, and Unreal Engine 4.x (Usually latest).

### Install Unreal

1. Download Unreal Engine.
  
2. Register and log in on the Unreal website: [https://accounts.unrealengine.com/login/index](https://accounts.unrealengine.com/login/index)
  
3. Download the Epic Games Launcher: [https://www.unrealengine.com/dashboard](https://www.unrealengine.com/dashboard)

4. Open the Epic Games Launcher.

5. Select the **Unreal Engine** tab, and **Library** from the left-hand navigation bar.

6. Select **+Add Versions**.

7. Select the most recent version of the [SDK](https://www.unrealengine.com/marketplace/en-US/playfab-sdk).

### Install the PlayFab Plugin into your engine

Use the following steps to ensure you've properly installed the PlayFab Plugin.

1. In the Epic Games launcher, go to the **Marketplace** and Search for the **PlayFab SDK**.

  ![Unreal Engine Marketplace](media/uemk-001.jpg)

2. Select the **PlayFab SDK**, then **Free**, and **Install to Engine**.

  ![Install to engine](media/uemk-install-to-engine.png)

3. Confirm your version and select **Install**.

    ![Pick version again](media/uemk-version-again.png)

4. Select the **Launch** button, and run Unreal Engine.

5. Select all the options as seen here:  **New Project** tab, **C++** sub-tab, **No Starter Content**.

      ![Create project settings](media/uemk-create-project-settings.jpg)

6. Now select **Create Project** with these options.

7. Enable the PlayFab Plugin.

  ![Enable plugin](media/uemk-enable-plugin.jpg)

The PlayFab Installation is complete!

## Set up your first Blueprint call

This section provides the minimum steps to make your first PlayFab Blueprint call. Confirmation is done via an on-screen debug print.

1. Select **Open level Blueprint**.

  ![Open level Blueprint](media/uemk-open-lv-bp.jpg)

2. Use the existing "Event BeginPlay" node, and build the  structure shown below.

  ![Event Begin Play](media/uemk-login-bp.png)

> [!NOTE]
> **Title ID** is the default, and should be unique to your game, which we call a title. You can apply any ID you want, but you must use that ID when you make PlayFab API calls.

3. **Save** the Blueprint, and close the Blueprint Editor window.

4. **Save** the level.

## Finish and execute with Blueprint

1. Push the **Play** button.

2. When you execute this program, you should get the output shown below.

  ![Blueprint log success](media/uemk-log-success.png)

3. Congratulations, you made your first successful API call!

4. Select any key to close.

## Set up your first C++ call

This section will provide the minimum steps to make your first PlayFab API call. Confirmation happens through a debug print in the Output Log.

1. Open your new project

2. Create a new actor called **LoginActor**, and place it in the scene. Creating the new LoginActor should automatically open Visual Studio, with `LoginActor.cpp` and `LoginActor.h` available to edit.

  ![New Actor C++](media/new-actor-cpp.png)

3. Under **Solution Explorer** -> **Games/YourProjectName/Source**, find and open **YourProjectName.Build.cs**.

4. Add the following line:

`PrivateDependencyModuleNames.AddRange(new string[] { "PlayFab", "PlayFabCpp", "PlayFabCommon" });`

5. Replace the contents of `LoginActor.h` with the code shown below.

```cpp
#pragma once

#include "GameFramework/Actor.h"

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"

#include "PlayFab.h"
#include "Core/PlayFabError.h"
#include "Core/PlayFabClientDataModels.h"

#include "LoginActor.generated.h"


UCLASS()
class ALoginActor : public AActor
{
    GENERATED_BODY()
public:
    ALoginActor();
    virtual void BeginPlay() override;
    void OnSuccess(const PlayFab::ClientModels::FLoginResult& Result) const;
    void OnError(const PlayFab::FPlayFabCppError& ErrorResult) const;

    virtual void Tick(float DeltaSeconds) override;
private:
    PlayFabClientPtr clientAPI = nullptr;
};
```

6. Replace the contents of `LoginActor.cpp` with the following code.

```cpp

#include "LoginActor.h"

#include "Core/PlayFabClientAPI.h"

ALoginActor::ALoginActor()
{
    PrimaryActorTick.bCanEverTick = true;
}

void ALoginActor::BeginPlay()
{
    Super::BeginPlay();

    clientAPI = IPlayFabModuleInterface::Get().GetClientAPI();
    clientAPI->SetTitleId(TEXT("144"));

    PlayFab::ClientModels::FLoginWithCustomIDRequest request;
    request.CustomId = TEXT("GettingStartedGuide");
    request.CreateAccount = true;

    clientAPI->LoginWithCustomID(request,
        PlayFab::UPlayFabClientAPI::FLoginWithCustomIDDelegate::CreateUObject(this, &ALoginActor::OnSuccess),
        PlayFab::FPlayFabErrorDelegate::CreateUObject(this, &ALoginActor::OnError)
    );
}

void ALoginActor::OnSuccess(const PlayFab::ClientModels::FLoginResult& Result) const
{
    UE_LOG(LogTemp, Log, TEXT("Congratulations, you made your first successful API call!"));
}

void ALoginActor::OnError(const PlayFab::FPlayFabCppError& ErrorResult) const
{
    UE_LOG(LogTemp, Error, TEXT("Something went wrong with your first API call.\nHere's some debug information:\n%s"), *ErrorResult.GenerateErrorReport());
}

void ALoginActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
}
```

6. Run the Unreal Editor (**Debug** -> **Start Debugging**).

## Finish and execute with C++

Earlier, you created a level with a `LoginActor` entity already placed in the world.

1. Load this level.

2. Press **Play**. You will immediately see the following in the output log:

`LogTemp: Congratulations, you made your first successful API call!`

  ![C++ log verify](media/ue-log-verify.png)

3. Select any key to close.

## Deconstruct the Blueprint example

This optional last section describes each part of the blueprints above, in detail.

### Event BeginPlay

This is an Unreal node that exists by default for a level blueprint. It triggers the nodes following it immediately, when the level is loaded.

### Set PlayFab Settings

Use this to set the `titleId`. Other keys can be set here too, but for this guide, you only need to set `titleId`.

Every PlayFab developer creates a title in Game Manager. When you publish your game, you must code that `titleId` into your game. This lets the client know how to access the correct data within PlayFab. For most users, just consider it a mandatory step that makes PlayFab work.

### Make the LoginWithCustomID request

Most PlayFab API methods require input parameters, and those input parameters are packed into a request object.

Every API method requires a unique request object, with a mix of optional and mandatory parameters.  

- For the `LoginWithCustomIDRequest` object, there is a mandatory parameter of `CustomId` - which uniquely identifies a player - and `CreateAccount`, which allows the creation of a new account with this call.
  
### Login with Custom ID

This begins the async request to `LoginWithCustomID`.

- For login, most developers will want to use a more appropriate login method. See the [PlayFab Login documentation](xref:titleid.playfabapi.com.client.authentication) for a list of all login methods, and input parameters. Common choices are:
  - [LoginWithAndroidDeviceID](xref:titleid.playfabapi.com.client.authentication.loginwithandroiddeviceid)
    - [LoginWithIOSDeviceID](xref:titleid.playfabapi.com.client.authentication.loginwithiosdeviceid)
    - [LoginWithEmailAddress](xref:titleid.playfabapi.com.client.authentication.loginwithemailaddress)

### The left-side blueprint pins  

- **Blue: Request** - For every PlayFab API blueprint, this must always receive from a paired Make Request blueprint node.

- **Red: "On Success" and "On Failure"** - You can drag an un-bound red marker to empty space, to create a new custom event for this action. One of those events, according to circumstances, is then invoked when the async-call returns.

- **Cyan: Custom Data** - Custom Data is just a relay. That object is passed un-touched into the red custom events. This isn't terribly useful for blueprints, but it's very useful when invoking API calls directly from C++ (Advanced topic: won't be covered in this guide).

### The right-side blueprint pins

- **White** - The unlabeled first exec pin is executed immediately as the API call is queued (response does not exist yet) - Do not use this pin!

- **White** - The second exec pin is labeled **On PlayFab Response**, and is executed after the async remote call has returned. Use this to trigger logic that needs to wait or use the Response.

- **Blue: Response**  
  - This is a JSON representation of the result.  
- The `OnSuccess` pin provides a properly typed object with the correct fields pre-built into the blueprint.
  - This JSON field is an older pin which is only maintained for legacy.

- **Cyan: Custom Data** - Same as **Custom Data** above.

- **Maroon: Successful**
  - Legacy boolean which indicates how to safely unpack the legacy Response pin.
  - Again, it's better to use the red `OnSuccess` and `OnFailure` pins.
  
### OnLoginSuccess and OnLoginFail

The names of these modules are optional, and should be different for every API call.

Described above, they attach to the red pins of PlayFab API calls, and allow you to process success and failure for those calls.

### The OnSuccess/Result pin

The result pin will contain the requested information, according to the API called.

### Break PlayFab Result

(Not displayed, the only valid connection for the `OnSuccess`/`Result` pin).

If you drag the `Result` pin from `OnSuccess`, it'll create a `Break-Result` blueprint. This blueprint is used to examine the response from any API call.

### The OnFailure/Error pin

- Always connects to a Break PlayFabError blueprint.
- Contains some information about why your API call failed.

### Why API calls fail (In order of likelihood)

- `PlayFabSettings.TitleId` is not set. If you forget to set `titleId` to your title, then nothing will work.  

- Request parameters. If you have not provided the correct or required information for a particular API call, then it will fail.

- See `error.errorMessage`, `error.errorDetails`, or `error.GenerateErrorReport()` for more info.  

- Device connectivity issue. Cell-phones lose/regain connectivity constantly, and so any API call at any time can fail randomly, and then work immediately after. Going into a tunnel can disconnect you completely.  

- PlayFab server issue. As with all software, there can be issues. See our [release notes](../../release-notes/index.md) for updates.

- The internet is not 100% reliable. Sometimes the message is corrupted or fails to reach the PlayFab server.

- If you are having difficulty debugging an issue, and the information within the error information is not sufficient, please visit us on our [forums](https://community.playfab.com/index.html)
  
### Prints and Append nodes

- Just part of the example, giving you some on-screen feedback about what's happening.

- Most examples will extract and utilize the data, rather than just printing.

## Deconstruct the C++ code example

This optional last section describes the code in this project line by line.

### GettingStartedUeCpp.Build.cs

- To reference code from a plugin in your project, you have to add the plugin to your code dependencies. The Unreal build tools do all the work, if you add the "PlayFab" string to your plugins.
  
### LoginActor.h

- includes
  - The `LoginActor` includes are default includes that exist for the template file before we modified it.
  - The PlayFab includes are necessary to make PlayFab API calls.
  - UCLASS `ALoginActor`
  - Most of this file is the default template for a new actor; the only exceptions to this are:
- `OnSuccess` and `OnError`
  - These are the asynchronous callbacks that will be invoked after PlayFab `LoginWithCustomID` completes.
- `PlayFabClientPtr` clientAPI
  - This is an object that lets you access the PlayFab client API.

### LoginActor.cpp

- Most of this file is the default template for a new actor; the only exceptions to this are:
- `clientAPI = IPlayFabModuleInterface::Get().GetClientAPI();`
  - This fetches the clientAPI object from the PlayFab plugin, so you can make API calls with it.
  - `clientAPI->SetTitleId(TEXT("xxxx"));`
    - Every PlayFab developer creates a title in Game Manager. When you publish your game, you must code that titleId into your game. This lets the client know how to access the correct data within PlayFab. For most users, just consider it a mandatory step that makes PlayFab work.
  - `PlayFab::ClientModels::FLoginWithCustomIDRequest request;`
    - Most PlayFab API methods require input parameters, and those input parameters are packed into a request object.
    - Every API method requires a unique request object, with a mix of optional and mandatory parameters.
    - For LoginWithCustomIDRequest, there is a mandatory parameter of CustomId, which uniquely identifies a player and CreateAccount, which allows the creation of a new account with this call.
  - `clientAPI->LoginWithCustomID(request, {OnSuccess delegate}, {OnFail delegate});`
    - This begins the async request to `LoginWithCustomID`, which will call `LoginCallback` when the API call is complete.
    - For login, most developers will want to use a more appropriate login method.
    - See the [PlayFab Login documentation](xref:titleid.playfabapi.com.client.authentication) for a list of all login methods, and input parameters. Common choices are:
      - [LoginWithAndroidDeviceID](xref:titleid.playfabapi.com.client.authentication.loginwithandroiddeviceid)
      - [LoginWithIOSDeviceID](xref:titleid.playfabapi.com.client.authentication.loginwithiosdeviceid)
      - [LoginWithEmailAddress](xref:titleid.playfabapi.com.client.authentication.loginwithemailaddress)
  - {OnSuccess delegate}: `PlayFab::UPlayFabClientAPI::FLoginWithCustomIDDelegate::CreateUObject(this, &ALoginActor::OnSuccess)`
    - combined with: `void ALoginActor::OnSuccess(const PlayFab::ClientModels::FLoginResult& Result) const`
    - These create a UObject callback/delegate which is called if your API call is successful.
    - An API Result object will contain the requested information, according to the API called.
      - `FLoginResult` contains some basic information about the player, but for most users, login is simply a mandatory step before calling other APIs.
  - {OnFail delegate} `PlayFab::FPlayFabErrorDelegate::CreateUObject(this, &ALoginActor::OnError)`
    - combined with: `void ALoginActor::OnError(const PlayFab::FPlayFabError& ErrorResult) const`
    - API calls can fail for many reasons, and you should always attempt to handle failure.
    - Why API calls fail (In order of likelihood)
      - `PlayFabSettings.TitleId` is not set. If you forget to set `titleId` to your title, then nothing will work.
      - Request parameters. If you have not provided the correct or required information for a particular API call, then it will fail. See `error.errorMessage`, `error.errorDetails`, or `error.GenerateErrorReport()` for more info.
      - Device connectivity issue. Cell-phones lose/regain connectivity constantly, and so any API call at any time can fail randomly, and then work immediately after. Going into a tunnel can disconnect you completely.
      - PlayFab server issue. As with all software, there can be issues. See our [release notes](../../release-notes/index.md) for updates.
      - The internet is not 100% reliable. Sometimes the message is corrupted or fails to reach the PlayFab server.
    - At this time, the PlayFab Unreal C++ SDK maintains state with static variables which are non atomic and are not guarded by synchronization techniques. For this reason, we recommend limiting PlayFab calls to within the main Unreal thread.
    - If you are having difficulty debugging an issue, and the information within the error information is not sufficient, please visit us on our [forums](https://community.playfab.com/index.html).

## Upgrading to the Unreal Marketplace Plugin

The [Unreal Marketplace Plugin - Upgrade Tutorial](unreal-marketplace-plugin-upgrade-tutorial.md) will step you through upgrading your project from either the PlayFab Unreal C++ SDK or the PlayFab Unreal Blueprint SDK, to the new PlayFab Unreal Marketplace Plugin.
