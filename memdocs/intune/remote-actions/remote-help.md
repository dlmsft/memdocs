---
# required metadata

title: Remotely assist users that are authenticated by your organization. 
description: With the remote help app, provide remote assistance to authenticated users who also run the remote help app.
keywords:
author: brenduns
ms.author: brenduns
manager: dougeby
ms.date: 11/24/2021
ms.topic: how-to
ms.service: microsoft-intune
ms.subservice: remote-actions
ms.localizationpriority: high
ms.technology:
ms.assetid: 

# optional metadata

#ROBOTS:
#audience:
#ms.reviewer: nehashah
ms.suite: ems
search.appverid: MET150
#ms.tgt_pltfrm:
ms.custom: intune-azure
ms.collection: M365-identity-device-management
---
 
# Use remote help with Intune and Microsoft Endpoint Manager

In [public preview](../fundamentals/public-preview.md), *remote help* is an application that works with Intune and enables your information and front-line workers to get assistance when needed over a remote connection. With this connection, your support staff can remote connect to the user's device. During the session they can view the devices display and if permitted by the device user, take full control. Full control enables a helper to directly make configurations or take actions on the device.

This feature applies to:  
- Windows 10/11

In this article, we'll refer to the users who provide help as *helpers*, and users that receive help as *sharers* as they share their session with the helper. Both helpers and sharers sign in to your organization to use the app. It's through your Azure Active Directory (Azure AD) that the proper trusts are established for the remote help sessions.

Remote help uses Intune role-based access controls (RBAC) to set the level of access a helper is allowed. Through RBAC, you determine which users can provide help and the level of help they can provide.

The remote help app is available from Microsoft to install on both devices enrolled with Intune and devices that aren’t enrolled. The app can also be deployed through Intune to your managed devices.

## Remote help capabilities and requirements

The Remote help app supports the following capabilities:

- **Enable remote help for your tenant** – By default, Intune tenants aren't enabled for remote help. If you choose to turn on remote help, its use is enabled tenant-wide. Remote help must be enabled before users can be authenticated through your tenant when using remote help.

- **Use remote help with unenrolled devices** – Disabled by default, you can choose to allow help to devices that aren't enrolled with Intune.

- **Requires Organization login** - To use remote help, both the helper and the sharer must sign in with an Azure Active Directory (Azure AD) account from your organization. You can’t use remote help to assist users who aren’t members of your organization.

- **Compliance Warnings** - Before connecting to a user's device, a helper will see a non-compliance warning about that device if it’s not compliant with its assigned policies. This warning doesn’t block access but provides transparency about the risk of using sensitive data like administrative credentials during the session.

  Unenrolled devices always report as non-compliant. This is because until a device enrolls with Intune it can’t receive policies from Intune and as such is unable to establish its compliance status.

- **Role-based access control** – Admins can set RBAC rules that determine the scope of a helper’s access, like:
  - The users who can help others and the range of actions they can do while providing help, like who can run elevated privileges while helping.
  - The users that can only view a device, and which can request full control of the session while assisting others.

- **Elevation of privilege** - When needed, a helper with the correct RBAC permissions can interact with the  UAC prompt on the sharer's machine to enter credentials. For example, your Help Desk employees might enter their administrative credentials to complete an action on the sharer’s device that requires administrative permissions.

- **Monitor active remote help sessions, and view details about past sessions** – In the Microsoft Endpoint Manager admin center you can view reports that include details about who helped who, on what device, and for how long. You’ll also find details about active sessions.

  For unenrolled devices, auditing and reporting about the remote help sessions is limited.

## Prerequisites

- [Intune subscription](../fundamentals/licenses.md)
- Windows 10/11
- Devices must install the *remote help* app. Device users can download the app directly from the Microsoft. See [Install and update remote help](#install-and-update-remote-help)


> [!NOTE]
> Remote help has the following limitations:  
>
> - Remote help is not supported on GCC High tenants.
> - You cannot establish a remote help session from one tenant to a different tenant.
> - May not be available in all markets or localizations.

### Network considerations

Remote help communicates over port 443 (https) and connects to the Remote Assistance Service at `https://remoteassistance.support.services.microsoft.com` by using the Remote Desktop Protocol (RDP). The traffic is encrypted with TLS 1.2.

Both the helper and sharer must be able to reach these endpoints over port 443:

| Domain/Name                       | Description                                           |
|-----------------------------------|-------------------------------------------------------|
| \*.support.services.microsoft.com | Primary endpoint used for the remote help application    |
| \*.resources.lync.com             | Required for the Skype framework used by remote help |
| \*.infra.lync.com                 | Required for the Skype framework used by remote help |
| \*.latest-swx.cdn.skype.com        | Required for the Skype framework used by remote help |
| \*.login.microsoftonline.com       | Required for logging in to the application (AAD). Might not be available in preview in all markets or for all localizations.   |
| \*.channelwebsdks.azureedge.net    | Used for chat services within remote help        |
| \*.aria.microsoft.com             | Used for accessibility features within the app    |
| \*.api.support.microsoft.com       | API access for remote help                          |
| \*.vortex.data.microsoft.com      | Used for diagnostic data                                |
| \*.channelservices.microsoft.com  | Required for chat services within remote help        |

### Data and privacy

Microsoft logs a small amount of session data to monitor the health of the remote help system. This data includes the following information:

- Start and end time of the session
- Errors arising from remote help itself, such as unexpected disconnections
- Features used inside the app such as view only, annotation, and session pause

Remote help logs session details to the Windows Event Logs on the device of both the helper and sharer. Microsoft can't access a session or view any actions or keystrokes that occur in the session. Microsoft cannot access a session or view any actions or keystrokes that occur in the session.

The helper and sharer both see the following information about the other individual, taken from their organizational profiles:

- Their organization profile picture (if present)
- Company name
- Verified domain
- First and Last name
- Job title

Microsoft does not store any data about either the sharer or the helper for longer than three days.

## Install and update remote help

Remote help is available as  download from Microsoft and must be installed on each device before that device can be used to participate in a remote help session.

When an update to remote help that is required, users are prompted to install that version of remote help when the app opens. The same process to download and install the initial remote help can be used to install the updated version. There's no need to uninstall the previous version before installing the updated version.

- Intune admins can download and deploy the app to enrolled devices. For more information about app deployments, see [Install apps on Windows devices](../apps/apps-windows-10-app-deploy.md#install-apps-on-windows-10-devices).
- Individual users who have permissions to install apps on their devices can also download and install remote help.

**Download remote help**: Download the latest version of remote help direct from Microsoft at [aka.ms/downloadremotehelp](https://aka.ms/downloadremotehelp).


## Configure remote help for your tenant

To configure your tenant to support remote help, review and complete the following tasks.

### Task 1 – Enable remote help

1. Sign in to [Microsoft Endpoint Manager admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant administration** > **Connectors and tokens** > **Remote help (preview)**.

2. On the **Settings** tab:
   1. Set **Enable remote help** to **Enabled** to allow use of remote help.  By default, this setting is *Enabled*.
   2. Set **Allow remote help to unenrolled devices** to **Enabled** if you want to allow this option. By default, this setting *Not allowed*.

3. Select **Save**.

### Task 2 – Configure permissions for remote help

The following Intune RBAC permissions manage use of the remote help app. Set each to *Yes* to grant the permission:

- Category: **Remote help app**
- Permissions:
  - **Take full control** – Yes/No
  - **Elevation** – Yes/No
  - **View screen** – Yes/No

By default, the built-in **Help Desk Operator** role sets all of these permissions to **Yes**. You can use the built-in role or create custom roles to grant only the remote tasks and remote help app permissions that you want different groups of users to have. For more information on using Intune RBAC, see [Role-based access control](../fundamentals/role-based-access-control.md).

> [!IMPORTANT]  
> When a remote help session ends where a helper that has the *Elevation* permission set to Yes also uses *Full control*, the sharer is automatically signed out of their device. Therefore, it is important for helpers who are granted these permissions to understand the impact this action can have. The sharer should plan to save any active work to prevent a loss of unsaved work.

### Task 3 – Assign user to roles

After creating the custom roles that you'll use to provide different users with remote help permissions, assign users to those roles.

1. Sign into [Microsoft Endpoint Manager admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant administration**  > **Roles** >  and select a role that grants remote help app permissions.

2. Select **Assignments**  > **Assign** to open **Add Role Assignment**.

3. On the *Basics* page, enter an **Assignment name** and optional **Assignment description**, and then choose **Next**.

4. On the **Admin Groups** page, select the group that contains the user you want to give the permissions to. Choose **Next**.

5. On the **Scope (Groups)** page, choose a group containing the users/devices that the member above will be allowed to manage. You also can choose all users, or all devices. Choose **Next** to continue.

   >[!IMPORTANT]
   >If a sharer’s device isn’t in the scope of a helper, that helper cannot provide assistance.

6. On the **Review + Create** page, when you're done, choose **Create**. The new assignment is displayed in the list of assignments.

## How to use remote help

The use of remote help depends on whether you're requesting help or providing help.

### Request help

To request help, you must reach out to your support staff to request assistance. You can reach out through a call, chat, email, and so on, and you'll be the sharer during the session. Be prepared to enter a security code that you'll get from the individual who is assisting you. You'll enter the code in your remote help instance to establish a connection to the helpers instance of remote help. 

> [!TIP]  
> To avoid an unexpected loss of work, plan to save your active work before a remote help session ends. This is because when a remote help session ends where a helper that has the *Elevation* permission set to Yes also uses *Full control*, you are signed out of your device.

As a sharer, when you’ve requested help and both you and the helper are ready to start:

1. Start the remote help app on the device and sign-in to authenticate to your organization. The device might not need to be enrolled to Intune if your administrator allows you to get help on unenrolled devices.

2. After signing into the app, get the security code from the individual assisting you and enter that code below *Get Help*, and then select **Submit**.

3. After submitting the security code from the helper, the helper will see information about you including your full name, job title, company, profile picture, and verified domain. As the sharer, your app displays similar information about the helper.

   At this time, the helper might request a session with full control of your device or choose only screen sharing. If they request full control, you can select the option to *Allow full control* or choose to *Decline the request*. Full control must be established before the help session starts. If full control is desired after the sessions starts, both users must disconnect and restart the remote help session.

4. After establishing they type of session (full control or screen sharing), the session is established and the helper can now assist resolving any issues on the device.

   During assistance, helpers that have the *Elevation* permission can enter local admin permissions on your shared device. *Elevation* allows the helper to run executable programs or take similar actions when you lack sufficient permissions.

5. After the issues are resolved, or at any time during the session, both the sharer or helper can end the session. To end the session, select **Leave** in the upper right corner of the remote help app. Upon the end of a session, the sharer is automatically signed out of their device as a security precaution to ensure all connections between the devices close.

### Provide help  

> [!TIP]  
> Plan to have the sharer save any active work before a remote help session ends to avoid an unexpected loss of work. This is because when a remote help session ends where a helper that has the *Elevation* permission set to Yes also uses *Full control*, the sharer is signed out of their device to ensure any elevated permissions are cleared from the device.

As a helper, after receiving a request from a user who wants assistance through the remote help app:

1. Start the remote help app on your device and sign-in to authenticate to your organization.

2. After signing into the app, under *Give help* select **Get a security code**. Remote help generates a security code that you’ll share with the person who has requested assistance. They'll enter this code in their instance of remote help to establish a connection to your remote help instance.

3. After the sharer enters the security code, as the helper you'll see information about the sharer, including their full name, job title, company, profile picture, and verified domain. The sharer will see similar information about you.

   At this time, you can request a session with full control of the sharers device or choose only screen sharing. If you request full control, the sharer can choose to *Allow full control* or to *Decline the request*. Full control must be established before the help session starts. If full control is desired after the sessions starts, both users must disconnect and restart the remote help session.

4. After establishing that the session will use a shared display or full control, remote help will display a **Compliance Warning* if the sharers device fails to meet the conditions of its assigned compliance policies.

   During assistance, helpers that have the *Elevation* permission can enter local admin permissions on your shared device. *Elevation* allows the helper to run executable programs or take similar actions when you lack sufficient permissions.

5. After the issues are resolved, or at any time during the session, both the sharer or helper can end the session. To end the session, select **Leave** in the upper right corner of the remote help app. Upon the end of a session, the sharer is automatically signed out of their device as a security precaution to ensure all connections between the devices close.

## Monitoring and reports

You can monitor the use of remote help from within Microsoft Endpoint Manager.

1. Sign into the [Microsoft Endpoint Manager admin center](https://go.microsoft.com/fwlink/?linkid=2109431) and go to **Tenant admin** > **Connectors and tokens** > **Remote help (preview)**.

2. On the Monitor tab, you’ll see a count of active sessions and historical data about past sessions.

3. On the Remote help sessions tab, you’ll see the records of past sessions, including:
   - The helper (Provider ID) and sharer (Recipient ID) of each session.
   - The device that received assistance.
   - The start and end time of the Remote Assistance session.

## Next steps

[Get support in Microsoft Endpoint Manager admin center](../../get-support.md)
