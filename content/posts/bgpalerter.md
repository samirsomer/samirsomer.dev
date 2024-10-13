---
title: "Monitor your BGP Activities with BGPAlerter"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["bgp", "monitoring", "technical"]
author: "Me"
categories: ["Networking"]
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

The internet is a collection connected networks, at the heart of it lies BGP as the backbone for exchanging BGP is at the heart of how the internet runs and operates. as such, monitoring your BGP network is an important task to ensure no configuration or malicious attacks are present at your network. [BGPAlerter](https://github.com/nttgin/BGPalerter) is a tool developed by NTT that monitors eBGP in real-time with the ability to send alerts to multiple notification channels. with BGPAlerter you can monitor you network for any of the following scenarios:

-   Prefixes loses visibility.
-   Prefixes is hijacking.
-   Invalid RPKI announcements (e.g., not matching prefix length).
-   Prefixes announcements not covered by ROAs.
-   Expiring ROAs.
-   RPKI Trust Anchors malfunctions
-   your AS is announcing a new prefix that was never announced before.
-   an unexpected upstream (left-side) AS appears in an AS path.
-   an unexpected downstream (right-side) AS appears in an AS path.
-   one of the AS paths used to reach your prefix matches a specific condition defined by you.

## ## Installing BGPAlerter

BGPAlerter can be installed either by using the per-compiled binaries, compiling from source, or using Docker. for simplicity We’ll be using the Binary version running on Linux Ubuntu 18.04.

First we are going to download the binary release from [here.](https://github.com/nttgin/BGPalerter/releases)

```bash
mkdir /opt/bgpalerter 
cd /opt/bgpalerter 
wget https://github.com/nttgin/BGPalerter/releases/download/v1.29.0/bgpalerter-linux-x64
```
Next, simply change the permission to make it executable and run the binary
```bash
chmod +x bgpalerter-linux-x64 
mv bgpalerter-linux-x64 bgpalerter 
./bgpalerter
```

## Configuring BGPAlerter
First time running bgrpalerter you will be with a configuration wizard that will walk you through the configuration, you will be asked to:

-   Autonomous Systems Numbers you want to monitor, you can monitor multiple ASNs separated by commas
-   decide if you want to receive alerts when new prefixes are announced
-   decide if you want to receive alerts if a new upstream/downstream appears in the path

```bash
The file prefixes.yml cannot be loaded. Do you want to auto-configure BGPalerter? 
Yes 
Which Autonomous System(s) you want to monitor? (comma-separated, e.g., 2914,3333) 
15706 
Do you want to be notified when your AS is announcing a new prefix? 
Yes 
Do you want to be notified when a new upstream AS appears in a BGP path? 
Yes 
Do you want to be notified when a new downstream AS appears in a BGP path? 
Yes
```
After completing the wizard, BGPAlerter will create the config file "config.yml" and "prefixes.yml"

### Prefixes Configuration

the prefixes.yml file will contains all the prefixes belonging to the ASNs you are monitoring with additional attributes

| Field         | Description                               | Expected Type                   |Required |
|--------------|-------------------------------------------|------------------------------------|-----|
| asn | The expected origin AS(es) of the prefix. | integer or an array of integers. | Yes |
| description | A description that will be reported in the alerts. | string | Yes |
|ignoreMorespecifics| Prefixes more specific of the current one will be excluded from monitoring.| boolean| Yes |
|ignore|Exclude the current prefix from monitoring. Useful when you are monitoring a prefix and you want to exclude a particular sub-prefix.| boolean | - |
|includeMonitors|The list of monitors you want to run on this prefix. If this attribute is not declared, all monitors will be used. Not compatible with excludeMonitors.|An array of strings (monitors name according to config.yml)|-|
|excludeMonitors|The list of monitors you want to exclude on this prefix. Not compatible with includeMonitors. Use monitors name attributes, as defined in the monitor list in config.yml.|An array of strings (monitors name according to config.yml)|-|
|path|A list path matching rules, read more here.||-|
|group|The name of the group that will receive alerts about this monitored prefix. See here.|string|-|

below is an example prefixes configuration:
```yaml
10.0.0.0/24: 
	description: Customer1 
	asn: 
		- 65536 
	ignoreMorespecifics: false 
	ignore: false 
	group: group1 
192.168.0.0/24: 
	description: No description provided (No ROA available) 
	asn: 
		- 65536 
	ignoreMorespecifics: false 
	ignore: false 
	group: group2
```

### BGPAlerter Config.yml
All of BGPalerter configuration is stored in config.yml, the main sections you will be configuring are:

-   Monitors: analyze the data flow and produce alerts. Different monitors try to detect different issues.
-   Reports: configure your notification channels.


### Defining a Notification channel
by default alerts will appear on logs/report.log, alternatively additional methods of notification can be configured such as email, slack, telegram, alerta, kafka, ...etc.

below sample shows configuring email alerts.
```YAML
  - file: reportEmail
    channels:
      - hijack
      - newprefix
      - visibility
      - path
      - misconfiguration
      - rpki
      - roa
    params:
      showPaths: 5 # Amount of AS_PATHs to report in the alert
      senderEmail: bgpalerter@thenetmechanic.com
      # BGPalerter uses nodemailer.
      # The smtp section can be configured with all the parameters available at https://nodemailer.com/smtp/
      # the following are just the most useful one
      smtp:
        host: mail.thenetmechanic.com
        port: 25
        secure: false # If true the connection will use TLS when connecting to server. If false it will be still possible doing connection upgrade via STARTTLS
        ignoreTLS: true # If true TLS will be completely disabled, including STARTTLS. Set this to true if you see certificate errors in the logs.
        auth:
          user: samir@thenetmechanic.com
          pass: <email_password>
          type: login
        tls:
          rejectUnauthorized: true  # Reject unauthorized certificates
      notifiedEmails:
        default:
          - samir@thenetmechanic.com
```

the email you will receive will look like
```BASH
type:announcement timestamp:1643956781131 prefix:2a00:5884::/32 peer:124.0.0.3 path:[1,2,3,204092] nextHop:124.0.0.3 aggregator:null 

DETAILS: 
------------------------------------------------------ 
Monitored prefix: 10.0.0.0/24 
Prefix Description: Customer1 
Usually announced by: 65536 
Event type: monitor-passthrough 
Now announced by: 65538 
Now announced with: 2a00:5884::/32 
When event started: 2022-02-04 06:39:41 UTC 
Last event: 2022-02-04 06:39:41 UTC 
Detected by peers: 1 
See in BGPlay: https://bgplay.massimocandela.com/?resource=0.0.0.0/0&ignoreReannouncements=true&starttime=1643956481&endtime=1643956781&rrcs=0,1,2,5,6,7,10,11,13,14,15,16,18,20&type=bgp 

Top 1 most used AS paths: 
2,3,204092
```
## Monitoring BGPAlerter
You can monitor your BGPAlerter process to make sure the service is always up using the "uptimeAPI". this enables retrieving the current status of BGPAlerter through API.

To configure uptimeAPI in the config.yml
```YAML
processMonitors: 
 - file: uptimeApi 
   params: useStatusCodes: true
```
The API is reachable at `http://localhost:8011/status` and provides a summary of the status of various components of BGPalerter. If any of the components is having a problem, the attribute `warning` is set to true.

Below is example of of the API output:

```BASH
samir@tnm01:~$ curl -s http://localhost:8011/status | jq
{
  "warning": false,
  "connectors": [
    {
      "name": "ConnectorRIS",
      "connected": true
    }
  ],
  "rpki": {
    "data": true,
    "stale": false,
    "provider": "rpkiclient"
  }
}
samir@tnm01:~$
```

You can change the port or the IP address from the localhost in the config.yml
```YAML
rest: 
  host: localhost 
  port: 8011
```

In conclusion, BGP monitoring is a critical task for ensuring the security and stability of your network. BGPAlerter offers a powerful and customizable solution for real-time monitoring of BGP activity, allowing you to detect issues such as prefix hijacking, invalid announcements, or unexpected changes in AS paths. By following the straightforward installation and configuration steps outlined, you can set up BGPAlerter to receive alerts via various notification channels, ensuring that you're always informed about potential network vulnerabilities. Proactively monitoring BGP can help safeguard your infrastructure from both misconfigurations and malicious attacks.