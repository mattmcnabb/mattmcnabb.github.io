---
layout: post
title: Disable Exchange Online Mailboxes for Active Users
categories: [Office 365]
author: Matt McNabb
comments: true
---

[Inactive]: https://technet.microsoft.com/en-us/library/dn144876(v=exchg.150).aspx
[Community]: https://community.office365.com/en-us/f/148/p/444880/1137949#1137949
[LitHold]: https://technet.microsoft.com/en-us/library/dn743673%28v=exchg.150%29.aspx?f=255
[InPlace]: https://technet.microsoft.com/en-us/library/dd979797(v=exchg.150).aspx
[LicenseBlog]: /Office-365-Licensing_1
[Rule]: https://technet.microsoft.com/en-us/library/jj919238%28v=exchg.150%29.aspx?f=255&MSPPError=-2147217396

This will be a quick post to detail the steps I took to resolve an issue in Exchange Online where we had a very specific use case for mailbox compliance.

We have a type of user that only has need of a mailbox for a certain period of time, and once this time is passed then according to our policy access to that mailbox will be removed. However, other services in Office 365 such as Onedrive for Business and the Office Pro Plus subscription will need to be retained. Our compliance policy also dictates that the mailbox data will need to be retained for an extended period of time. We user Litigation Hold to achieve this retention.

When a mailbox is on Litigation Hold and the corresponding user is deleted, the mailbox is converted to "Inactive" and all it's data is retained. The [Inactive][guidance provided by Microsoft] for this all centers around employees who are leaving your organization which is why the trigger for converting to an inactive mailbox is the deletion of the user.

In our case, however, we don't intend to delete these users but we would like to remove the mailboxes. This turned out to be a bit trickier than I would have thought. I asked about how best to achieve our goal [Community][here] but did not get a full answer, so I set out to experiment about how to achieve this. Here are the steps I followed to disable mailboxes for active Office 365 users:

### 1. Set the mailbox on Litigation Hold

This step is important because deactivated mailboxes can only be restored for up to 30 days after a user is deleted. In addition, this retains mailbox data even after a user has deleted it, ensuring that you can discover messages for legal compliance cases.

Instructions for setting a mailbox to litigation hold can be found [LitHold][here].

> NOTE: You can use [InPlace][In-Place Hold] instead of Litigation Hold, but in our case I stick with Litigation Hold because it's very simple to use and the settings belong to the mailbox. In-Place holds are a bit more complex and are an entity all their own.

### 2. Remove the Exchange and Exchange Archiving Licenses

This is how we remove the user's access to their Exchange Online mailbox. Once this is removed all entry points into Exchange Online will be hidden from the user, and sign-in will fail.

You can remove the Exchange Online license using the Office 365 Admin Center, or you can use Powershell. To learn more about managing Office 365 user licenses with PowerShell, see my [LicenseBlog][blog series] on the topic. The key is that you should only remove the Exchange and Exchange Archiving service plans and leave any other applicable service plans in place.

### 3. Disable send and receive

This one was tricky. Although removing the Exchange license for a user removes their access to their mailbox, the mailbox itself can still receive email messasges. While this is not a huge deal, I didn't want to create the false impression that a deprovisioned mailbox was still in active use, so I decided to leverage a transport rule to handle this.

To create a transport rule that prevents sending and receving for these mailboxes, first you'll need to create a new distribution group and add your mailboxes to it. I called mine "DisabledMailboxes". Once you've done this, create a new transport rule with the following conditions:

1. If the message is sent to a member of the group "DisabledMailboxes@domain.onmicrosoft.com"

2. reject the message and include the explanation 'The mailbox you attempted to send to no longer exists on the server!' with the status code '5.7.1'

3. and stop processing more rules

If you need more information creating transport rules, go [Rule][here].

### 4. Delete users

After the first 3 steps above are completed, what you're left with is an active user with a disconnected mailbox that is on Litigation Hold. Ultimately users will be deleted when they are no longer a part of your organization. When this happens Exchange will take over and convert the active mailbox to an inactive one. The mailbox data will the be retained until the end of the Litigation Hold duration.

I hope you've found this article useful - thanks for reading!