---
title: Detecting Orthrus and Typhon
date: 2021-08-05 06:00:00 +01000
tags: [Blue, Detection, Black Hat]
image:
  src: /assets/img/detecting-orthrus-and-typhon/blackhat2021_logo.png
  width: 501
  height: 211
  alt: Black Hat USA 2021 Logo
---

On 5th August 2021, we ([Calum @\_calumhall](https://twitter.com/_calumhall) and [Luke @rookuu\_](https://twitter.com/rookuu_)) presented at Black Hat USA, with our talk "Come to The Dark Side We Have Apples: Turning macOS Management Evil". The talk discusses methods of using and abusing Jamf and MDM in an offensive operation, and specifically relevent to this post - as initial access implants. 

To facilite the research, we created two [Mythic](https://github.com/its-a-feature/Mythic) agents. In this post, we wanted to outline some of the ideas we've had to detection the agents. Most of the detection opportunities below are targeted towards the installation of the agents, as opposed to the malicious activity post compromise. This is because by nature, the agents are designed to blend in with the leigimate management software extant within the target's estate.

## Orthrus

![orthrus logo](/assets/img/detecting-orthrus-and-typhon/orthrus.svg){: width="300" height="300" }
_Orthrus: A Mythic agent utilising mdmclient and APNs._

Orthrus is a implant and C2 mechanism that uses Apple's MDMCLient and Push Notification Service. Effectively, the operator coherses a target to enroll into their malicious MDM server, gaining (some) control over the device. Full documentation, installation and usage instructions can be found [here](https://github.com/MythicAgents/orthrus).

### Malicious .mobileconfig Downloaded from the Internet.

In order to compromise a target with Orthrus, a malicious .mobileconfig file must be generated, and installed as a MDM profile on the target device. This is typically performed by sending the file to the target, via email or web download, along with some pretext about $COMPANY IT needing to fix their device. This action would be logged when the file is created, for example, using [ESF: File Create](https://developer.apple.com/documentation/endpointsecurity/es_event_create_t) 

**Detection:** Raise alert for files ending in `.mobileconfig` that have been downloaded from the internet or send via email.

### Device Enrolled in a (Different) MDM Server.

The exact nature of this idea will depend on whether you run MDM already within your organisation. If you don't, then naturally devices that are suddenly enrolled into an MDM server is cause for concern. Similarly, if devices are enrolled in MDM servers OTHER than yours, this is indicative of a threat actor using similar techniques to those used by Orthrus.

**Decection:**  Use something like [OSQuery](https://github.com/osquery/osquery/blob/master/specs/darwin/managed_policies.table) to check for profiles on a device, to see if/what MDM profile is installed. 

### Uninstall MDM Message

Similar to the previous idea, if an organisation is already using MDM then they are able to tell if a device has uninstalled the profile. When an MDM profile is uninstalled, it sends a `CheckOut` message. See the [MDM Protocol Reference (Page 13)](https://developer.apple.com/business/documentation/MDM-Protocol-Reference.pdf).

**Detection:** Use existing MDM solution to monitor for `CheckOut` messages.

## Typhon

![typhon logo](/assets/img/detecting-orthrus-and-typhon/typhon.svg)
_Typhon: A Mythic agent designed to exploit Jamf managed devices._

Typhon is a Mythic agent that was developed to enable device takeover attacks against Jamf managed workstations. This agent & profile combination enables an attacker to imitate the behaviour of a Jamf server within macOS environments, and as such operate in a covert manner.

### Alteration of the Jamf Configuration

Typhon was designed as a means of taking over control of Jamf enrolled devices - without introducing any custom code. Jamf operates on a client server model where the client side agent relies on a configuration file to determine which server to communicate with. The typhon agent provides the user with a custom Jamf configuration file, that once placed within `/Library/Preferences/com.jamfsoftware.jamf.plist` on the device will cause the endpoint to communicate with the specified Mythic server.

**Detection:** It is highly unusual for any process other than Jamf to interact with the Jamf configuration file (with the exception of a few default system utilities). Alerts should be raised for any alterations made to this file by non-standard processes. Additionally, it may be possible using tools such as osquery to monitor for changes to the configuration file's hash value.

### Device Health Check

This type of device takeover attack currently prevents the compromised device from checking in to the legitimate Jamf server.

**Detection:** It may be possible to implement a device health check within your environment to identify devices that have not checked in for a prolonged period of time. 

### Code Execution

Whilst the Jamf agent does legitimately execute bash commands within the majority of environments. It should not be assumed that any commands performed by this binary are legitimate. As attacks like this have demonstrated, the Jamf agent may be abused to perform malicious activity on behalf a malicious user.

**Detection:** Ensure that the Jamf binary is not whitelisted from current device detections. 
