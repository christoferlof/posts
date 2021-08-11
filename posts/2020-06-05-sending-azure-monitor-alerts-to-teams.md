---
title: Sending Azure Monitor alerts to Microsoft Teams
layout: post
tags: azure
excerpt: Leveraging Microsoft Teams as a target for Azure Monitor alerts is an easy and efficient approach in handling issues in your Azure solutions
---
Leveraging Microsoft Teams as a target for Azure Monitor alerts is an easy and efficient approach in handling issues in your Azure solutions. Posting the alert to a Teams channel ensures that the team is notified and also serves as a good place for discussing the various situations as they arise. 

First step is to create a channel in Teams to which you want the alerts to be sent. Then, [configure a webhook for that channel](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) and copy the webhook URL. As part of creating the webhook, you'll be able to configure an icon for it, so why not use one of the [service icons like alert or monitor](https://www.microsoft.com/en-us/download/details.aspx?id=41937)?

Next, you need to create the webhook as a target in an [Monitor Action Group](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/action-groups).
When configuring the Action Group, create a new target of type webhook and add the URL you copied from Teams in the step above. 

The last step is to setup your alert and select the newly created Alert group as a target when the alert triggers. The important piece here is to specify the payload to be sent, otherwise Teams won't understand it and will not push it to the channel. Teams use the [Office Connector schema](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using) for it's incoming webhooks.

```json
{
    "@context": "http://schema.org/extensions",
    "@type": "MessageCard",
    "themeColor": "CC4216",
    "title": "#alertrulename",
    "text": "#alertrulename returned #searchresultcount records which exceeds the threshold of #thresholdvalue .",
    "summary": "Query: #searchquery",
    "potentialAction": [{
        "@type": "OpenUri",
        "name": "See details in AppInsights",
        "targets": [{
            "os": "default",
            "uri": "#linktosearchresults"
        }]
    }],
    "sections": [{
        "facts": [{
            "name": "Severity:",
            "value": "#severity"
        },
        {
            "name": "Query:",
            "value": "#searchquery"
        },
        {
            "name": "ResultCount:",
            "value": "#searchresultcount"
        },
        {
            "name": "Search Interval StartTime:",
            "value": "#searchintervalstarttimeutc"
        },
        {
            "name": "Search Interval End time:",
            "value": "#searchintervalendtimeutc"
        },
        {
            "name": "AppInsights Application ID:",
            "value": "#applicationid"
        }]
    }]
}
```

The above json payload will create something simlar to below in your Teams channel

![Azure Monitor Alert in Teams](/assets/article_images/teams-alert.png)