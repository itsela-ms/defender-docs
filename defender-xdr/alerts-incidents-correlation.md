---
title: Alerts, incidents, and correlation in Microsoft Defender XDR
description: Learn how alerts and incidents are correlated in Microsoft Defender XDR.
ms.service: defender-xdr
f1.keywords: 
  - NOCSH
ms.author: yelevin
author: yelevin
ms.localizationpriority: medium
manager: dansimp
audience: ITPro
ms.collection: 
  - m365-security
  - usx-security
  - tier1
ms.topic: conceptual
search.appverid: 
  - MOE150
  - MET150
ms.date: 05/30/2024
appliesto: 
- Microsoft Defender XDR 
- Microsoft Sentinel in the Microsoft Defender portal
---

# Alerts, incidents, and correlation in Microsoft Defender XDR

In Microsoft Defender XDR, ***alerts*** are signals from a collection of sources that result from various threat detection activities. These signals indicate the occurrence of malicious or suspicious events in your environment. Alerts can often be part of a broader, complex attack story, and related alerts are aggregated and correlated together to form ***incidents*** that represent these attack stories.

***Incidents*** provide the full picture of an attack. Microsoft Defender XDR's algorithms automatically correlate signals (alerts) from all Microsoft security and compliance solutions, as well as from vast numbers of external solutions through Microsoft Sentinel and Microsoft Defender for Cloud. Defender XDR identifies multiple signals as belonging to the same attack story, using AI to continually monitor its telemetry sources and add more evidence to already open incidents.

Incidents also function as "case files," providing you a platform for managing and documenting your investigations. For more information about incidents' functionality in this regard, see [Incident response in the Microsoft Defender portal](incidents-overview.md).

[!INCLUDE [unified-soc-preview](../includes/unified-soc-preview.md)]

Here is a summary of the main attributes of incidents and alerts, and the differences between them:

**Incidents:**

- Are the main "unit of measure" of the work of the Security Operations Center (SOC).
- Display the broader context of an attack&mdash;the **attack story**.
- Represent "case files" of all the information needed to investigate the threat and the findings of the investigation.
- Are created by Microsoft Defender XDR to contain at least one alert, and in many cases, contain many alerts.
- Trigger automatic series of responses to the threat, using [automation rules](/azure/sentinel/automate-incident-handling-with-automation-rules?tabs=onboarded), [attack disruption](automatic-attack-disruption.md), and [playbooks](/azure/sentinel/automation/automate-responses-with-playbooks).
- Record all activity related to the threat and its investigation and resolution.

**Alerts:**

- Represent the individual pieces of the story that are essential to understanding and investigating the incident.
- Are created by many different sources both internal and external to the Defender portal.
- Can be analyzed by themselves to add value when deeper analysis is required.
- Can trigger [automatic investigations and responses](m365d-autoir.md) at the alert level, to minimize the potential threat impact.

## Alert sources

Microsoft Defender XDR alerts are generated by many sources:

- Solutions that are part of Microsoft Defender XDR
  - Microsoft Defender for Endpoint
  - Microsoft Defender for Office 365
  - Microsoft Defender for Identity
  - Microsoft Defender for Cloud Apps
  - The app governance add-on for Microsoft Defender for Cloud Apps
  - Microsoft Entra ID Protection
  - Microsoft Data Loss Prevention

- Other services that have integrations with the Microsoft Defender security portal
  - Microsoft Sentinel
  - Non-Microsoft security solutions that pass their alerts to Microsoft Sentinel
  - Microsoft Defender for Cloud

Microsoft Defender XDR itself also creates alerts. With Microsoft Sentinel onboarded to the [unified security operations platform](microsoft-sentinel-onboard.md), Microsoft Defender XDR's correlation engine now has access to all the raw data ingested by Microsoft Sentinel. (You can find this data in **Advanced hunting** tables.) Defender XDR's unique correlation capabilities provide another layer of data analysis and threat detection for all the non-Microsoft solutions in your digital estate. These detections produce Defender XDR alerts, in addition to the alerts already provided by Microsoft Sentinel's analytics rules.

When alerts from different sources are displayed together, each alert's source is indicated by sets of characters prepended to the alert ID. The [**Alert sources**](investigate-alerts.md#alert-sources) table maps the alert sources to the alert ID prefix.

## Incident creation and alert correlation

When alerts are generated by the various detection mechanisms in the Microsoft Defender security portal, as described in the previous section, Defender XDR places them into new or existing incidents according to the following logic:

| Scenario | Decision |
| -------- | -------- |
| The alert is sufficiently unique across all alert sources within a particular time frame. | Defender XDR creates a new incident and adds the alert to it. |
| The alert is sufficiently related to other alerts&mdash;from the same source or across sources&mdash;within a particular time frame. | Defender XDR adds the alert to an existing incident. |

The criteria Microsoft Defender uses to correlate alerts together in a single incident are part of its proprietary, internal correlation logic. This logic is also responsible for giving an appropriate name to the new incident.

## Incident correlation and merging

Microsoft Defender XDR's correlation activities don't stop when incidents are created. Defender XDR continues to detect commonalities and relationships between incidents, and between alerts across incidents. When two or more incidents are determined to be sufficiently alike, Defender XDR merges the incidents into a single incident.

### How does Defender XDR make that determination?

Defender XDR's correlation engine merges incidents when it recognizes common elements between alerts in separate incidents, based on its deep knowledge of the data and the attack behavior. Some of these elements include:

- Entities&mdash;assets like users, devices, mailboxes, and others
- Artifacts&mdash;files, processes, email senders, and others
- Time frames
- Sequences of events that point to multistage attacks&mdash;for example, a malicious email click event that follows closely on a phishing email detection.

### When are incidents *not* merged?

Even when the correlation logic indicates that two incidents should be merged, Defender XDR doesn't merge the incidents under the following circumstances:

- One of the incidents has a status of "Closed". Incidents that are resolved don't get reopened.
- The two incidents eligible for merging are assigned to two different people.
- Merging the two incidents would raise the number of entities in the merged incident above the maximum allowed.
- The two incidents contain devices in different [device groups](/defender-endpoint/machine-groups) as defined by the organization. <br>(This condition is not in effect by default; it must be enabled.)

### What happens when incidents are merged?

When two or more incidents are merged, a new incident is not created to absorb them. Instead, the contents of one incident are migrated into the other incident, and the incident abandoned in the process is automatically closed. The abandoned incident is no longer visible or available in Microsoft Defender XDR, and any reference to it is redirected to the consolidated incident. The abandoned, closed incident remains accessible in Microsoft Sentinel in the Azure portal. The contents of the incidents are handled in the following ways:

- Alerts contained in the abandoned incident are removed from it and added to the consolidated incident.
- Any tags applied to the abandoned incident are removed from it and added to the consolidated incident.
- A **`Redirected`** tag is added to the abandoned incident.
- Entities (assets etc.) follow the alerts they're linked to.
- Analytics rules recorded as involved in the creation of the abandoned incident are added to the rules recorded in the consolidated incident.
- Currently, comments and activity log entries in the abandoned incident are *not* moved to the consolidated incident.

To see the abandoned incident's comments and activity history, open the incident in Microsoft Sentinel in the Azure portal. The activity history includes the closing of the incident and the adding and removal of alerts, tags, and other items related to the incident merge. These activities are attributed to the identity *Microsoft Defender XDR - alert correlation*.

## Manual correlation

While Microsoft Defender XDR already uses advanced correlation mechanisms, you might want to decide differently whether a given alert belongs with a particular incident or not. In such a case, you can unlink an alert from one incident and link it to another. Every alert must belong to an incident, so you can either link the alert to another existing incident, or to a new incident that you create on the spot.

[!INCLUDE [Microsoft Defender XDR rebranding](../includes/defender-m3d-techcommunity.md)]

## Next steps

Read more about incidents, investigation, and response: [Incident response in the Microsoft Defender portal](incidents-overview.md)

## See also

- [The power of incidents in Microsoft Defender XDR](https://techcommunity.microsoft.com/t5/microsoft-defender-xdr-blog/the-power-of-incidents-in-microsoft-365-defender/ba-p/3515483)
- [Inside Microsoft Defender XDR: Correlating and consolidating attacks into incidents](https://www.microsoft.com/en-us/security/blog/2020/07/09/inside-microsoft-threat-protection-correlating-and-consolidating-attacks-into-incidents/)
