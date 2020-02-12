---
title: You are operating in a split-domain (hybrid) topology when set call queue
description: Discusses that you receive a "You are operating in a split-domain (hybrid) topology" warning message when you configure an Office 365 call queue in the Skype for Business admin center. Provides a workaround.
author: simonxjx
manager: dcscontentpm
localization_priority: Normal
search.appverid: 
- MET150
audience: ITPro
ms.prod: skype-for-business-itpro
ms.topic: article
ms.author: v-six
ms.custom: CSSTroubleshoot
appliesto:
- Skype for Business
---

# "Call Queue Agent transfers to other tenant users fail.

## Symptoms

After you set an Office 365 call queue in the Teams admin center, and you add a distribution group, transfers from agents to other tenant users fail. Skype for Business server logs will display following error:

# "SIPPROXY_E_EPROUTING_MSG_INVALID_EXT_SPLIT_DOMAIN"


## Cause

This issue occurs because your tenant is configured by using a split domain with Office 365, the Resource Account configured for the Call Queue is a custom domain, and DNS federation server records for this domain resolve to on-premises Skype for Business server. When transfers are made the call will route to the on-premises server which needs to be configured correctly to allow this connection. 

## Resolution

To resolve this issue, follow these steps:

1. Make sure that the on-premises deployment for Skype for Business and Office 365 is configured correctly for hybrid voice.   
2. Enable your Edge server to accept calls that are routed from sipfed.resources.lync.com (part of a typical hybrid configuration).   
3. Configure a new hosting provider in your on-premises configuration that enables communication to the Azure services that function as part of the OrgAA and call queue infrastructure. To do this, run the following cmdlet in Remote PowerShell:

    ```powershell
    New-CSHostingProvider -Identity SFBOnlineResources -ProxyFqdn "sipfed.resources.lync.com" -Enabled $true -EnabledSharedAddressSpace $true -HostsOCSUsers $true -VerificationLevel AlwayVerifiable -IsLocal $false
    ```
4. Make sure that domain that's specified for the call queue is the sip domain of the agents. Also, make sure that all agents in the queue are in the same sip domain. Do not specify "contoso.mail.onmicrosoft.com" or "onmicrosoft.com." If the domain was selected incorrectly when you created the call queue, you must re-create the call queue or OrgAA by using the correct domain.   

## More Information

To verify which agents are available to receive calls among the agents who are defined in the distribution list for a call queue, run the following cmdlet in Remote PowerShell:

```powershell
Get-CsHuntGroup
```

If the agents are not listed,follow these steps:

1. Verify that the agents are licensed for Skype for Business and Cloud PBX.   
2. Verify that the agents are Enterprise Voice-enabled. If they are assigned a PSTN Calling license, they will be Enterprise Voice-enabled already. If they have only a Cloud PBX license, you must manually enable them for Enterprise Voice. To determine whether it's necessary to enable the user for Enterprise Voice, run the following script from Remote PowerShell:

    ```powershell
    Get-CsOnlineUser -Identity <UPN or SIP Address> | Select SipAddress, EnterpriseVoiceEnable, HostedVoiceMail
    Set-CsUser -Identity <UPN or SIP Address> -EnterpriseVoiceEnable $True -HostedVoiceMail $True
    ```

For more information, see the following TechNet topics:

- [Plan for your Skype for Business Server 2015 deployment](https://technet.microsoft.com/library/dn951427.aspx)   
- [New-CsHostingProvider](https://technet.microsoft.com/library/gg398490.aspx)   
- [Set-CsHostingProvider](https://technet.microsoft.com/library/gg398532.aspx)   
- [Enable users for Enterprise Voice online and Cloud PBX Voicemail](https://technet.microsoft.com/library/mt455213.aspx)   
