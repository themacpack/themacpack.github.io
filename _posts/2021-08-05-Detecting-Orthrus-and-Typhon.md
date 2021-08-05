---
title: Detecting Orthrus and Typhon
date: 2021-08-05 06:00:00 +01000
tags: [Blue, Detection, Black Hat]
---

On 5th August 2021, we ([Calum (@_calumhall)](https://twitter.com/_calumhall) and [Luke (@rookuu_)](https://twitter.com/rookuu_) presented at Black Hat USA, with our talk "Come to The Dark Side We Have Apples: Turning macOS Management Evil". The talk discusses methods of using and abusing Jamf and MDM in an offensive operation, and specifically relevent to this post - as initial access implants. 

To facilite the research, we created two [Mythic](https://github.com/its-a-feature/Mythic) agents. In this post, we wanted to outline some of the ideas we've had to detection the agents. Most of the detection opportunities below are targeted towards the installation of the agents, as opposed to the malicious activity post compromise. This is because by nature, the agents are designed to blend in with the leigimate management software extant within the target's estate.

## Orthrus

![orthrus logo](/assets/img/detecting-orthrus-and-typhon/orthrus.svg)
_Orthrus: A Mythic agent utilising mdmclient and APNs._

Orthrus is a implant and C2 mechanism that uses Apple's MDMCLient and Push Notification Service. Effectively, the operator coherses a target to enroll into their malicious MDM server, gaining (some) control over the device. Full documentation, installation and usage instructions can be found [here](https://github.com/MythicAgents/orthrus).

### Malicious .mobileconfig Downloaded from the Internet.

In order to compromise a target with Orthrus, a malicious .mobileconfig file must be generated, and installed as a MDM profile on the target device. This is typically performed by sending the file to the target, via email or web download, along with some pretext about $COMPANY IT needing to fix their device. 

**Detection:** Raise alert for files ending in `.mobileconfig` that have been downloaded from the internet or send via email.

### Device Enrolled in a (Different) MDM Server.

### Uninstall MDM Message

## Typhon
