# Rodano roadmap

## Environment updates

- setup Sonar in the CI
- merge rodano-manager and migrator

## Documentation updates

- rework the Validation Summary (use the one from Castor as a model)

### Sample IQ
Create a sample IQ. Create a checklist for the creation of a new server (even if the server is created automatically with the Provisioner) and store the results the checks to prove the successful creation of the server. This checklist should contain the following:
- the study website is available at the right URL
- the certificate of the study is validated
- 3 unsuccessful attempts to connect to the server over SSH block the IP of the client for 10 minutes
- the webwatcher has been configured and send an e-mail of the study instance is down

## Technical updates

- fix mock email server in tests
- remove all TODOs and @Deprecated
- name Spring Boot HTTP parameters (as Spring will no longer infers the name of HTTP parameters from the name of the Java method parameters: as mentioned here https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x#parameter-name-retention)
- create a script in Java that runs all pending migrations and execute this script when the application starts
- in the script that checks the consistency of the database, the order of the columns must be checked, as it is important for jOOQ
- remove "ManagedException"
- use the DTO service to update business objects (as already done to update users)
- reorganize the application code packages
- put workflow buttons directly in the workflow panel
- improve test descriptions to improve readability of test reports
- streamline attributes and actions ids in the rule library
- delete the rule action "WRITE_TO_LOG" from "ScopeActionEntityDefiner"
- streamline the workflow statuses search API with the other search APIs
- streamline configuration entities comparators
- distinguish CREATE from SAVE in the DAO
- do no propagate technical exception from the service layer to the controllers (and the final user)
- rename "message" and "triggerMessage" in Workflow and WorkflowStatus to improve readability
- rename scope/scopeModel into node/nodeModel
- find a way to sort workflow widgets properly (sort is done of entity ids but entity labels are displayed in the UI)
- rename id to uuid in scope, event and dataset
- make the summary widget rely on the URL for the selection of the scope
- remove configurations flags for password length and strength

### Move business logic from controllers to services
This will allow making only the service layer transactional, which is logical since it's the layer that should be responsible for the data modification and controllers should just focus on receiving requests. Most of the controllers have already been cleaned up. The major ones that still need work are the "EproController" and "WorkflowStatusController".

### Allow to restore a removed dataset, without saving the form
Users who can see and restore data are different from users adding data. That's why users with the feature "ManageDeletedData" should be able to restore a dataset without saving the form.

### Rename audit trail API
The audit trail API represents the past versions of the object to which they are attached. They no longer represent the audit trail of a particular value, but the past versions of a particular object. It makes sense to rename the whole API to “versions” or something similar.

### Run all required migrations when the application starts
Currently, the "migration" profile of the backend application can only execute one migration at a time. The application must be able to run all the required migrations (SQL and Java) when it starts. The "migration" profile must be removed and the "migrator" application must be deleted.

### Streamline the way to retrieve the model of an entity
For every entity that has a model (field, dataset, event, scope), a method named "getModel" must return the model of the database entity. Each database entity must have a column in the database named "model" with the id of its model.

### Define a common e-mail template
E-mail notifications are sent to users on numerous subjects. There is no common template across the e-mails, which leads to some hideous texts. A common e-mail template to use across the system must be defined. The study-specific e-mails should also follow this template for consistency.

### Handle users who have multiple root scopes
The workflow and workflow summary widgets API return results associated with only one of the user’s root scopes. It must return results associated with all scopes on which the user has rights to.

### Rewrite storage of dates
Dates from the CRF must be stored using the ISO 8601 format instead of the Swiss format. A solution must be found to store partial dates.

### Improve AuditAction and DatabaseContext
An audit action must be inserted in the database only when audit trails will be attached to it, in order not to fill the database with useless audit actions. It's not possible to rely on the HTTP method to determine whether an audit action must be created or not because some POST/PUT/DELETE HTTP requests do not generate any audit trail.
On the opposite, a database context must be created for every HTTP request.

### Outsource tests of the application
Find an external developer to write unit and end-to-end tests of the application.

### Improve configuration semantic

- Menu
	- "Menu" should be renamed to "MenuOption" or equivalent (these are not menus, they are menu options)
	- rename "action" to something more explicit like "link" or "page"
	- use the "action page" to select the component in the frontend: it should probably be chosen from a list, since the possible values must be aligned with the frontend router
	- use the "action context" to specify what the generic components should display (specify what scope model the scope list component should display)
	- the "action context" should be a string instead of a list
	- remove the "action parameters" property
	- the embedded menu option layout should be attached directly to the MenuDTO object: it makes no sense to go and look for it again
	- remove the magic in the MenuDTO object construction: these implicit rules are difficult to parse and understand
- Widget
	- rename "CMSWidget" to "Widget"
	- the "parameters" map of the widgets makes them difficult to work with, since we have no idea what the "parameters" map might contain. Having specialised components seems like overkill though…
- Profile
	- clean the properties
- Workflow
	- rename actionId to creationActionId
- Workflow action
	- Rename "documentable"
- Dataset model
	- Rename "scopeDocumentation"
	- Get rid of "collapsed/expandedLabelPattern"
- Field model
	- Rename "forDisplay".
	- Should we introduce a time field?

## Business updates

### Restore 2-step authentication
TOTP 2-step must be restored and users should be allowed to enable 2FA in their "account" page. During the activation, the process is the following:
- generate and store a TOTP seed for the user
- present this seed to the user (barcode and text)
- ask the user to enter the current TOTP code to validate the activation
If enabled, ask for a TOTP code during the login of the user. If the login succeeds, store a footprint of the user client (trusted client) using their user agent and IP address. Do not re-ask the TOTP code for following login while the footprint does not change.
A solution should be found for user who lost the 2-step device (using "recovery codes" or asking the user to contact the support team).

### Display forms only once in the PDF documentation
If multiple events use the same form, the form should appear only once in the PDF documentation.

### Associate anonymous operations done via one-use code to the right user
Some operations are done is the system using one-use code sent by e-mail to users. These operations are logged in the system as anonymous operations (made by SYSTEM), but should be linked to the user who received the e-mail. These operations are:
- password reset
- e-mail verification
- e-mail update

### Make it possible to verify that the running code of a study is the good one
When launching the application, calculate checksums for each file used for a study (JSON configuration and Java plugins) and displays these checksums on the administration page. This would make it possible to check these checksums and ensure that they match what is expected.

### Replace the NOTIFY_RESOURCE_PUBLISHED feature by a trigger
Currently, an e-mail notification (programmed as part of the backend) is sent to profiles with the feature NOTIFY_RESOURCE_PUBLISHED.
Another approach would be to add a trigger "Resource published", allowing to configure the message to be sent. If this approach is chosen, it should be possible to retrieve the publication type, title and file name.

### Improve virtual scopes
Add the following features:
- be able to save a search of scopes as a virtual scope
- be able to create virtual scopes that are combinations (intersections or unions) or other scopes

### Delete useless labels in configuration
Almost every node in the configuration has 3 different fields used as labels (shortname, longname and description) and an id. When possible, keep only 2 or 3 fields:
- id
- label
- description

### Add action to set an event to "not done" (or missed)
We should rethink the way we handle "not done" events. A custom action in "EventAction" called "missedEvent" that sets the flag "not done" to true and manages everything required when an event is "not done" (expected flag, reset of next event dates, reset of forms and fields) should be created.

### Find a way to allow users who cannot save scope to transfer them
Typically, users who transfer patients (scopes) from one center (parent scope) to another don't have the right "WRITE" on patient. Find a way to allow this kind of users to transfer patients. Or find a way to allows 2 investigators who have the right to "WRITE" patients to initiate and validate a patient transfer.

### Offer pre-defined styles for form layouts
On some layouts with numerous fields, an alternate line colour is introduced to improve the readability of the CRF. Currently, the DMs are forced to overload the CSS and to specify the CSS class in the layout configuration.
Not only this is too complicated, it also goes against our idea of simplifying the layout classes and the eventual removal of the CSS overload file. An alternative would be to introduce this CSS class in the frontend itself and add an option to the layout config that specifies if the layout should use the alternate line styling.

### Handle the study logo properly
Each study has a logo. Currently, this logo is put somewhere by the DM in the custom code of the frontend and displayed in the header with custom CSS. The logo (svg, png or jpg) should be stored in a specific place in the study repository and displayed automatically in the header.
A solution could be to update the NGINX configuration to serve the logo. In the frontend, do a Javascript request to get the logo and display the logo only if the request succeeds.

### Move button related to workflow in the workflow panel
In the CRF, workflow status buttons are displayed in a menu in the miller’s columns. When there are many available workflow actions (on existing workflows or to create new workflows), this menu is confusing (especially when aggregated workflows and real workflows, potentially with the same name, are available for the same entity).
To solve this problem, the workflow buttons must be removed from the miller’s columns and added in the workflow panel, next to the presentation of the workflow. Workflows that can be created must be displayed in this panel, and the associated buttons to create these workflows must be displayed next to them. This workflow panel must then be used for scopes, visits and forms in the CRF and in the scope edition page.

### Integrate the current workflow status message directly into its database table
In its current state, only the original “trigger message” of a workflow status is present in the workflow status database table. The latest message of the workflow status must be retrieved in the audit trail table. This poses a problem since not all users who need to see the latest message have the right to access the audit trail table, and there is a need to display this message in several places, notably the widgets and the workflow status panel.
The latest message of the workflow status must be integrated directly into the workflow status database table.

### Create a raw text widget
Replace the welcome text widget by a raw text widget that can display free text.

### Hard-code columns of workflow widgets
It’s currently possible to choose the columns that will be displayed in each workflow widget. These columns should be hard-coded, depending on the entity on which the workflow is attached.

All widgets must have the following columns:
- workflow state
- workflow date

For workflows attached to scopes, the additional columns should be
- parent scope code
- scope code

For workflows attached to events, the additional columns should be
- event label
- event date

For workflows attached to fields, the additional columns should be
- field label

### Make field model id globally unique
Currently, field model IDs are unique among their document model. Make them globally unique. The goal is the following:
- match CDISC that has globally unique ID
- be able to retrieve the model of a field without fetching its dataset (to retrieve its document model)

### Define and calculate workflow status aging
For workflow status, calculate the duration between the date of last status and now. It must then be possible to categorize workflow status based on their age (and configure the different categories). It should be possible to display this age in workflow widgets and to display distribution of age.

### CRF versioning
The design of the CRF is based on the protocol of the study. During a study, amendments can be made to its protocol with consequences on the design and behavior of the CRF. The following elements can be updated:
- possible values of an attribute
- visibility criteria
- validators
- constraints on cells and layouts
- layout of pages
- events (scheduling and name)?

To implement this, the following configuration nodes must be updated:
- Field models (with their possible values and validators)
- Form models
- Event models
These configuration nodes, which can be modified to adapt to a new version of the study protocol, are called "versionable" nodes. Instances of these nodes, will be duplicated when a new version of the CRF is released.

Description of the version
A new configuration node must be created, named "version". A version consists of:
- an id (string)
- a name (string)
- a set of constraints having the couple "visit, scope" that describe when the version must be used
- on the study, a new parameter must be added to choose the default version.

In the configurator, all versions of "versionable" nodes must appear as a group. In the backend, all instances of the "versionable" nodes must be linked to a version using a new column "version" in the database.

#### Open question
- when will the node be linked to a version? Using rules? At creation?
- what happens if a scope or an event changes version? (reset all fields, forms)?
- in the export, should versioned fields be exported in the same column?

## Module updates

### Configurator

- restore the study creation wizard
- restore the form model creation wizard
- attach the field model wizard to the form model entity
- handle localization with a dedicated XML file managed by the configurator
- accept enums as entity properties
- use "string" and "number" instead of "STRING" and "NUMBER" to describe types in enums and entities (to match Typescript types)
- refresh the entity icons
- do not compress the configuration before sending it over HTTP (but keep the "weird" characters check)
- in tests, use XPath selectors that do not depend on the position of elements in the DOM tree (rely only on text content and/or text attributes)
- add JSDoc everywhere and generate a documentation using the JSDoc command line (jsdoc xxx.js)
- enforce code format (JavaScript, HTML and CSS) using VSCode repository configuration and ESLint
- store configuration version and date in entity templates
- create a simple mode and advanced mode to help users understand the application
- create a production mode that does not allow modifying entity ids
- create modules (CSS, HTML and JavaScript) in dedicated files for every tool and reports
- display the theme that match the OS theme
- use inputs and datalists instead of selects in rule edition form
- improve breadcrumbs for Layout, Column and Cell entities
- stash configuration before any update to be able to rollback modifications
- sort field models in the visibility criterion edition form
- prefix rule condition ids with "condition" (from #11 to #condition11) to match DOM id requirements
- handle rights when copy-pasting nodes
- allow to dimension cells that contain radio buttons

### Frontend

- remove all "UntypedFormGroup"
- streamline "mails" and "scopes" pages by displaying the search form and the result when the page loads
- in tables, attach the link used to browse a specific item to the name of the item instead of the whole row

### ePro

- convert the app into a webapp with a full offline mode, so that the can fill out forms offline
- split the difference and stock the user form completion time and the server reception time
- implement the dataset synchronisation when the app goes back online
- implement field dynamism (hide/show fields based on previous field answer)

#### Notify users when new events are available
In the current version of the application, the list of the available events is never refreshed automatically. This is a problem because:
- if a user keeps the app open for a long time, he will not see the events that become available (because of the scheduling of the events and the clock ticking, an event may become available).
- a new event may be created directly in the backend (by another user and with a rule), and the ePro app will not be aware of this new event until a request is made to the server

To remedy this issue:
- set up a cron in the ePro app that calls the server regularly and check if there are new events to show to the user
- if this is the case, display a notification to the user asking him to refresh the list (swiping down the list), but do not refresh automatically the list (the UI must not be updated without an action from the user)

### Timeline

- handle wrongly configured periods that are outside the global timeline range
- type the configuration (using the Reviver to transform dates and configuration objects)

## Charts 
### Backend 
- `ChartDataRepository`: Implementing a rights check, so the logged-in user can only see data of the scopes he can access. 
- Clean up existing code from the previous HighChart library. 
- Enable Audit Trails for the charts so changes to the chart configuration are available to look up somewhere. 
- `ChartDTO`: Category, RequestParameters and States are subclasses. We should consider moving them to their own class. 
- `ChartController`: Instead of having both a getBenchmarkData and getData API endpoint, we should merge them and make parameters required by the benchmarking page optional. 
- Right now, every chart parameter is sent via URL. This could lead to problems if we e.g. have many categories, which could lead to reaching the maximum limit.
- `ChartRepository`: We should use Jooq's multiset feature instead of having multiple round trips to fetch the categories, states and colors. 
- In `FieldModelCriterion.java`, there is a function `toCondition`. We already have something similar with JOOQTranslator::translate. It would nice to share this code here.

### Frontend
- We should have one form per configuration entity. 
- Manage labels properly for chart types and field models.
- use the new approach or NgClass
- use ReactiveForms
- Creating a ColorPicker component instead of relying in manually typing in colors as Hex-Strings. 
