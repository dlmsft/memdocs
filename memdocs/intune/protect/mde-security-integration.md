---
# required metadata

title: Use Microsoft Defender for Endpoint Security Configuration Management in Microsoft Endpoint manager
description: Use Intune profiles to manage security settings for Microsoft Defender for Endpoint on devices that register in your Azure Active Directory. 
keywords:
author: brenduns
ms.author: brenduns
manager: dougeby
ms.date: 11/22/2021
ms.topic: how-to
ms.service: microsoft-intune
ms.subservice: protect
ms.localizationpriority: medium
ms.technology:

# optional metadata
 
#ROBOTS:
#audience:
#ms.devlang:
ms.suite: ems
search.appverid: MET150
#ms.tgt_pltfrm:
ms.custom: intune-azure
ms.collection: M365-identity-device-management
ms.reviewer: mattcall

---

# Manage Microsoft Defender for Endpoint on devices with Microsoft Endpoint Manager

>[!Note]
> ***This feature is in public preview*** *and will roll out to tenants gradually over the next few weeks. You can confirm your tenant has received this capability when the relevant toggles show in both the Microsoft Endpoint Manager admin center and Microsoft Defender for Endpoint.*

With Microsoft Defender for Endpoint (MDE), you can now deploy security configurations from Microsoft Endpoint Manager directly to your onboarded devices without requiring a full Microsoft Endpoint Manager device enrollment. This capability is known as *Security Management for Microsoft Defender for Endpoint*. With this capability, devices that aren’t managed by a Microsoft Endpoint Manager service can receive security configurations for Microsoft Defender directly from Endpoint Manager.

When devices are managed through this capability:

- You use the Microsoft Endpoint Manager admin center to configure endpoint security policies for MDE and assign those policies to Azure AD groups
- Devices get the policies based on their Azure Active Directory device object. A device that isn’t already present in Azure Active Directory is joined as part of this solution
- When a device receives a policy, the Defender for Endpoint components on the device enforce the policy and report on the devices status. The device's status is available in the Microsoft Endpoint Manager admin center

This scenario extends the Microsoft Endpoint Manager Endpoint Security surface to devices that aren't capable of enrolling in Endpoint Manager. When a device is managed by Endpoint Manager (enrolled to Intune) the device won't process policies for Security Management for Microsoft Defender for Endpoint. Instead, use Intune to deploy policy for Defender to your devices.

:::image type="content" source="./media/mde-security-integration/endpoint-security-overview.png" alt-text="Conceptual diagram of the MDE-Attach solution." lightbox="./media/mde-security-integration/endpoint-security-overview.png":::

[!INCLUDE [Prerequisites](includes/security-config-mgt-prerequisites.md)]

## Monitor status

Status and reports for MDE policies are available from the policy node under Endpoint security in the Microsoft Endpoint Manager admin center.

Drill in to the policy type, Antivirus or Firewall, and then select the policy to view its status. Policies for MDE have a *Policy type* of either *Microsoft Defender Antivirus (Preview)* or *Microsoft Defender Firewall (Preview)*.

When you select a policy, you'll see information about the device check-in status, and can select:

- **View report** - View a list of devices that received the policy. You can select a device to drill in and see its per-setting status. You can then select a setting to view more information about it, including other policies that manage that same setting, which could be a source of conflict.

- **Per setting status** - View the settings that are managed by the policy, and a count of success, errors, or conflicts for each setting.

## Known limitations and considerations

### Co-existence with Microsoft Endpoint Configuration Manager
When using Configuration Manager, the best path for management of security policy is using the [Configuration Manager tenant attach](/mem/configmgr/tenant-attach/endpoint-security-get-started). In some environments it may be desired to use Security Management for Microsoft Defender. When using Security Management for Microsoft Defender with Configuration Manager, endpoint security policy should be isolated to a single control plane. Controlling policy through both channels will create the opportunity for conflicts and undesired results.

### Active Directory joined devices

Devices that are joined to Active Directory will use their **existing infrastructure** to complete Hybrid Azure Active Directory join. While the Defender for Endpoint component will start this process, the join action uses your Federation provider or Azure Active Directory Connect (AAD Connect) to complete the join. Review [Plan your hybrid Azure Active Directory join implementation](/azure/active-directory/devices/hybrid-azuread-join-plan) to learn more about configuring your environment.

To troubleshoot Azure Active Directory onboarding issues, see  [Troubleshoot Security Configuration Management Azure Active Directory onboarding issues](/microsoft-365/security/defender-endpoint/troubleshoot-security-config-mgt).

### Managing Security Configurations on domain controllers

Currently, devices are not supported to complete a Hybrid Join to Azure Active Directory. Since an Azure Active Directory trust is required, domain controllers aren't currently supported. We are looking at ways to add support in the future.

### Non-persistent VDI environments

Due to the potential effect on Azure Active Directory environments with respect to device lifecycle and service quota, we advise against testing the current installation files and builds shared in this public preview in a non-persistent VDI environment.

### Server Core installation

Due to the limited scope of Server core installations, these are not supported by Security Management for Microsoft Defender for Endpoint.

### PowerShell Constrained Language

Security Management for Microsoft Defender for Endpoint will not be able to apply security settings on devices if PowerShell Contrained Language mode is activated.

## Next steps

[Monitor Defender for Endpoint](../protect/advanced-threat-protection-monitor.md)  
