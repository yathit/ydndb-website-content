---
layout: userguide-article
title: "Leads"
introduction: "Sugar’s Leads module consists of individual prospects who may be interested in a product or service your organization provides. "
authors:
  - petelepage
article:
  written_on: 2014-04-30
  updated_on: 2014-04-30
  order: 1
collection: sugarcrm-records
---

{% wrap content%}

[Sugar’s Leads module](http://support.sugarcrm.com/02_Documentation/01_Sugar_Editions/04_Sugar_Professional/Sugar_Professional_7.2/Application_Guide/10_Leads/) consists of individual prospects who may be interested in a product or service your organization provides. Once the lead is qualified as a sales opportunity, leads can be converted into contacts, opportunities, and accounts. There are various ways you can create leads in Sugar such as via the Leads module, duplication, importing leads, etc. Once the lead record is created, you can view and edit information pertaining to the lead via the Leads record view. This documentation will go over the basics of the Leads module as well as the various options available in performing the actions related to the module.

## Creating a new Lead

A new Lead record can be created from [Sidebar](../sidebar/index.html) or [Context Widget](../context/index.html).
 
### Using Sidebar to create a new Lead

To create a new record using Sidebar, click "Create a new record" tab, {{ "plus" | svg_icon }}. ![Create a new lead](/imgs/sugarcrm/create-Leads.gif){: .callout}

A _new record form_ will appear on the panel (see in the figure). The _record form_ has a _header_ and _content_. Top header has an SugarCRM _record type icon_ on the left and a hamburger menu button on the right. Record type icon has two characters symbol of the record type, in this case, "Le" with record type color for Lead. 

The content panel consists input for record fields compartmentalized by their groups. If a field does not have group, it is placed under the default group. The label of a field can be seen by tooltip over the field input. Not all fields are displayed in the content panel. Field can be show or hide by changing setting in the "Fields..." menu from the hamburger menu. Some field are read only and cannot be change, but still be displayed.
 
Edit the fields as necessary and click _submit button_, {{ "check-circle" | svg_icon }}, on the header. The submit button appears only if content value(s) is dirty (change from initial value). Clicking submit button will send a new record create request to SugarCRM backend server. If success, header and content will be update with field values received from the server. Notice that header now should record label (Lead's full name) as web link to SugarCRM record view. You may use the web link to see record in SugarCRM web app.

### Using Context widget to create a new Lead

Creating a new Lead record inside Gmail [Context Widget](../context/index.html) is a little different, more convenient and smarter. ![Create a new lead](/imgs/sugarcrm/create-Leads-context.gif){: .callout}

When a new email thread is open, Gmail show _info column_ on the right side of the main panel. The top of Gmail info column is about _the most relevant contact_ panel. If there are more contacts involved in the email, additional contact link are provided under the most relevant contact. (Sometimes it may happen that the most relevant contact is not available for some email like system generated message). Yathit CRMinInbox context widget panel is injected just below the most relevant contact panel. Email address and its full name are taken to the context widget panel and retrieve from SugarCRM records. If any SugarCRM record is found matching the email address, the record is shown. Otherwise, it is hidden and only _search input_ is shown. 

To create a new Lead record, you can either type email address or full name to the search input and then click search button, {{ "search" | svg_icon }}, or press enter. You may also use auto suggested list in the search input. The new Lead record form will appear below the search input box with email or full name field filled. If social add-on is available, additional contact field values will be fill up from publicly available information from multiple social networks.

Edit the fields as necessary and click _submit button_, {{ "check-circle" | svg_icon }}, on the header.  
 
## Updating a Lead
 
Updating a record is similar to creating a record. You can either use [Sidebar](../sidebar/index.html) or [Context Widget](../context/index.html).
  
First search the record you want to edit. Click search button tab, {{ "magnifying-glass" | svg_icon }}, on the Sidebar. ![Edit a lead](/imgs/sugarcrm/edit-Leads.gif){: .callout}
 
Click edit menu item from the hamburger menu, {{ "menu" | svg_icon }}, on the right of record header to edit the record. 
 
Edit the fields in the _record content panel_ as necessary and click _submit button_, {{ "check-circle" | svg_icon }}, on the header. The submit button appears only if content value(s) is dirty (change from initial value). A message should appear to notify the record is updated. The record content panel is updated with field values received from the server.   

## Deleting a Lead

{% endwrap %}
