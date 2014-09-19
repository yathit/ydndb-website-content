---
layout: userguide-article
title: "Accounts"
introduction: "Sugar’s Accounts module consists of companies with whom your organization has a relationship and is generally seen as the hub for managing and analyzing your business’ interactions with each customer."
authors:
  - petelepage
article:
  written_on: 2014-04-30
  updated_on: 2014-04-30
  order: 3
collection: sugarcrm-records
---

{% wrap content %}

[Sugar’s Accounts module](http://support.sugarcrm.com/02_Documentation/01_Sugar_Editions/04_Sugar_Professional/Sugar_Professional_7.2/Application_Guide/12_Accounts/) consists of companies with whom your organization has a relationship and is generally seen as the hub for managing and analyzing your business’ interactions with each customer. There are various ways you can create accounts in Sugar such as via the Accounts module, duplication, importing accounts, etc. Once the account record is created, you can view and edit information pertaining to the account via the Accounts record view. Each account record may then relate to other Sugar records such as contacts, meetings, cases, opportunities, and many others as the customer’s relationship matures. This documentation will cover how to use the Accounts module as well as the various actions and options available from within the module.

## Creating a new Account

A new Account record can be created from [Sidebar](../sidebar/index.html) or [Context Widget](../context/index.html).
 
### Using Sidebar to create a new Account

To create a new record using Sidebar, click "Create a new record" tab, {{ "plus" | svg_icon }}.

A _new record form_ for Leads will appear on the panel. The _record form_ has a _header_ and _content_. Top header has an SugarCRM _record type icon_ on the left and a hamburger menu button on the right. Record type icon has two characters symbol of the record type, in this case, "Le" with record type color for Leads. Change to Accounts record type by selecting New > Accounts menu item from the _hamburger menu_. Notice that record type icon change to Accounts. 

The content panel consists input for record fields compartmentalized by their groups. If a field does not have group, it is placed under the default group. The label of a field can be seen by tooltip over the field input. Not all fields are displayed in the content panel. Field can be show or hide by changing setting in the "Fields..." menu from the hamburger menu. Some field are read only and cannot be change, but still be displayed.
 
Edit the fields as necessary and click _submit button_, {{ "check-circle" | svg_icon }}, on the header. The submit button appears only if content value(s) is dirty (change from initial value). Clicking submit button will send a new record create request to SugarCRM backend server. If success, header and content will be update with field values received from the server. Notice that header now should record label (Account's full name) as web link to SugarCRM record view. You may use the web link to see record in SugarCRM web app. 
 
## Updating a Account
 
Use [Sidebar](../sidebar/index.html) to update a Accounts record.
  
First search the record you want to edit. Click search button tab, {{ "magnifying-glass" | svg_icon }}, on the Sidebar.
 
Click edit menu item from the hamburger menu, {{ "menu" | svg_icon }}, on the right of record header to edit the record. 
 
Edit the fields in the _record content panel_ as necessary and click _submit button_, {{ "check-circle" | svg_icon }}, on the header. The submit button appears only if content value(s) is dirty (change from initial value). A message should appear to notify the record is updated. The record content panel is updated with field values received from the server.   

## Deleting a Account

To delete a record, search the record you want to delete. Click search button tab, {{ "magnifying-glass" | svg_icon }}, on the Sidebar.

Click delete menu item from the hamburger menu, {{ "menu" | svg_icon }}, on the right of record header to delete the record. 

A message should appear to notify the record is deleted.

## Searching Account records

Account records can be search on search tab, {{ "magnifying-glass" | svg_icon }}, on the Sidebar. You can search record fields by emails, name, id, description.


{% endwrap %}
