---
layout: post
title: Office 365 Preservation Policies, or A Recursive Nightmare
categories: [Office365]
author: Matt McNabb
comments: true
---

[Uservoice]: https://office365.uservoice.com/forums/273493-office-365-admin/suggestions/14093619-users-can-t-delete-folders-when-site-is-on-preserv
[Overview]: https://support.office.com/en-us/article/Overview-of-preservation-policies-9c3b1d52-40ce-4ba3-a520-9ae0be15538a?ui=en-US&rs=en-US&ad=US

This will be a short blog post that serves as a warning to any folks out there thinking of employing Office 365 Preservation Policies in their organization.

We recently decided to employ the new Office 365 Preservation Policies and we've become aware of a glaring issue that affects end users. It seems that when a Preservation Policy is applied to a Sharepoint Site, one unexpected affect is that folders can not be deleted if they contain any other items. The solution to this should be to delete items within folder first and then when it's empty, delete the folder.

The real problem arises when a folder contains a OneNote Notebook. Sharepoint sees the notebook as a folder that contains files and will not allow the user to delete them. Users in our organization essentially can't perform common folder cleanup tasks due to an administrative feature that according to it's description should only affect files after they are deleted. This is a big problem.

I have worked with Microsoft Support to troubleshoot the issue, and once we determined the scope of the problem they stated that the product team is aware and will be working to fix the problem. However, they are not able to provide any estimated delivery date for this fix. If you use Office 365 and this problem affects you, please vote up my [Uservoice item][Uservoice] here and let's get this fixed for our users!

In theory Preservation Policies will be a boon to those who are subject to document compliance policies, and I'm sure this will be worked out eventually, but not if Microsoft isn't made aware of how much of a negative affect this can cause!

For reference, you can learn more about Preservation Policies [here][Overview].

Thanks for reading!