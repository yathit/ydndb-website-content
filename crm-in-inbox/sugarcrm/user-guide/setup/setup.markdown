---
layout: userguide-article
title: "Setup"
description: "Setting up Yathit CRMinInbox for SugarCRM is following simple instruction, but there are some more tweak or get stuck."
introduction: "Setting up Yathit CRMinInbox for SugarCRM is following simple instruction, but there are some more tweak or get stuck."
article:
  written_on: 2014-04-30
  updated_on: 2014-04-30
  order: 2
collection: sugarcrm-setup
---


{% wrap content%}

## Install

Yathit CRMinInbox for SugarCRM is Chrome [Browser Extension](https://chrome.google.com/webstore/category/extensions) and hence works only with [Chrome Browser](https://www.google.com/chrome/browser/). Unlike [Chrome App](https://chrome.google.com/webstore/category/apps), Extension can, with user explicit permission, interact and enhance websites, such as [Gmail website](https://mail.google.com). User should conscious about giving permission to any extension not more than as necessary to perform the tasks required. Yathit CRMinInbox for SugarCRM require website data access to Gmail (https://mail.google.com), Yathit (https://www.yathit.com) and your SugarCRM websites.

Install Yathit CRMinInbox for SugarCRM by clicking the following button:

<div class="centered">
    <a id="install-sugarcrm" class="button--primary themed">Add to Chrome</a>
</div>

This will prompt permission grant dialog as follow:

![Install permission dialog](/imgs/sugarcrm/install-permission.jpg) 

Click "Add" to grant permission and install the extension.

Then visit to [Gmail](https://mail.google.com), you should see Yathit CRMinInbox sidebar panel tab appear on right side of Gmail website. Click the button for setting up account.

You can find installed extensions in [Chrome Extension Page](chrome://extensions/) ([screenshoot](/imgs/sugarcrm/chrome-extensions-page.tiff)). Extension page is used to uninstall the extension, force updating, view given permissions and visit to extension option page. For developer, there is a link to view browser console to debug extension background page.
  
Extension has a background page. It runs only when browser is opening [Gmail](https://mail.google.com) or SugarCRM website. The background page will inject CRMinInbox sidebar and context panel using extension [content script](https://developer.chrome.com/extensions/content_scripts) technology.   
 
## Login to Yathit server
 
Yathit CRMinInbox extension connect to Yathit server for accessing [Google services authorization tokens](https://developers.google.com/accounts/docs/OAuth2) and account auditing purposes. Yathit server uses Google account. You should login with same account as your Gmail account.

After login to the server, provide your Google Contact, Calender and Task data to Yathit server by clicking 'Grant.
 
If you sign in with [multiple accounts](https://support.google.com/accounts/answer/1721977) to Gmail, only the account that login to Yathit server will be active. You can still use Yathit CRMinInbox extension for read only mode.  

Login registration to Yathit server is free. You can use Yathit CRMinInbox extension for free, but functionality will be limited.
 
## Setting SugarCRM
 
Bring up [Setup wizard page](chrome-extension://{{ site.sugarcrm.extension_id }}/setup.html). The link is also available in extension sidebar and option page.
 
![SugarCRM setup](/imgs/sugarcrm/sugarcrm-setup.gif) 
 
 
Enter SugarCRM website URL, username and password in Section 3: SugarCRM login.
 
It will as to grant host access to your SugarCRM domain. Click 'Allow' to accept the requested permission.
 
The extension will synchronize (download) your SugarCRM data to the extension in background page. It will take a while to complete the database synchronization. During the time you can start using it.


{% endwrap %}