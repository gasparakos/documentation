:author: Miklós Bán
:date: 2026-08-08

Administrative settings
***********************

The project administration interface provides tools for configuring an
OpenBioMaps project, managing its users and data structures, and monitoring
project-related services. The pages available to an administrator depend on
the administrator's permissions, the project configuration, the installed
modules, and the server environment.

This page provides an overview of the administrative settings and tools. Some
settings affect access to project data or modify the underlying database.
Administrators should therefore review changes carefully and test them before
applying them to a production project.

For an overview of the project administration documentation, see
:doc:`Project administration <../admin_pages>`.


.. _administrative-access:

Administrative access
=====================

The **Administrative access** section allows project administrators to
delegate individual administrative functions to user groups. Each function
available through the project administration interface can be assigned to
one or more groups.

This provides fine-grained control over who can perform administrative
tasks. For example, a project could define the following groups:

* **User managers**, with access to user and group management;
* **Data curators**, with access to species names, attachments, and data
  management tools; and
* **Upload form editors**, with access to upload form management.

Grant only the permissions required for each administrative role. Functions
that change database structures, execute SQL, manage access rules, or edit
executable code should be restricted to trusted administrators.

.. TODO: Document every assignable administrative function and the
   permissions it grants. It should also be clarified whether permissions
   inherited through nested groups are evaluated recursively and whether a
   user needs to sign in again after their administrative permissions
   change.


.. _admin-sidebar-pins:

Pinning administration pages to the sidebar
===========================================

Every card on the administration page (the engine room) has a pin button.
Pinning a page adds it to the profile sidebar, under the same categories in
which it appears on the administration page. A page that belongs to several
categories has a single pin state and appears in each of them. Categories
without pinned pages are not shown. The card on the administration page remains
available.

Pins are personal: each administrator has their own list, which is saved with
the project options. Pinning a page never grants access to it, and pages that
the user cannot open are hidden even if they were pinned earlier. If the same
user changes the pins in two sessions at the same time, the last save wins.

.. _sql-columns-vs-openbiomaps-columns:

Database tables and columns
===========================

The **Database tables and columns** section is used to create and manage the
SQL tables, views, and columns associated with a project. Objects registered
through this interface are added to the OpenBioMaps metadata and can
therefore be made available to upload forms, queries, modules, and other
OpenBioMaps interfaces.

Tables and columns created directly through a standard SQL client are not
registered automatically. They must also be added to the relevant
OpenBioMaps metadata before they can be used through the web application.

.. TODO: Explain how an existing SQL table or view can be registered without
   recreating it. Document which metadata tables are modified by this
   interface and whether database objects created outside the interface can
   be imported safely.


Naming tables and columns
-------------------------

Use lowercase letters, numbers, and underscores for table and column names.
Avoid spaces, accented characters, quoted identifiers, and other special
characters. Names should be descriptive and should remain stable after forms,
queries, or modules begin using them.

A description (Table comment or Column comment) should be provided whenever a
table or column is created.
These descriptions form part of the project's metadata and help users
understand the meaning and intended use of the data.

.. TODO: Document the complete naming rules enforced by the interface,
   including maximum lengths, reserved names, schema handling, and whether a
   name may begin with a number.

.. _database-columns:

Registering available columns
-----------------------------

Administrators can select which columns are available when creating upload
forms and query interfaces. A column that exists in PostgreSQL but is not
registered as available will not automatically appear in these interfaces.

Columns can also be assigned semantic roles. These roles allow OpenBioMaps
and its modules to identify important fields without relying on a
project-specific column name. Depending on the project and installed
modules, roles may identify fields containing:

* a scientific name;
* an alternative taxon name;
* an observation date;
* a data collector;
* a location or geometry;
* a count of individuals;
* latitude and longitude values;
* a citation; or
* an attachment.

.. TODO: Provide a complete list of semantic roles and identify which core
   functions or modules use each role. Clarify whether more than one column
   can have the same role and whether one column can have multiple roles.

.. _column-type:

Column types
------------

The administrative interface provides the following documented column
types or semantic roles:

``Data``
   A general-purpose data column.

``Spatial Geometry``
   A geometry column used for maps and spatial operations.

``Scientific Species Name``
   A scientific-name column used by taxon-management functions.

``Alternative Names``
   An alternative-name column used by taxon-management functions.

``Date``
   A date or date-time column used by date filters.

``Number of Individuals``
   A numeric column used by summary functions.

``Latitude/Longitude``
   A coordinate column used to create spatial geometry.

``Citing``
   A citation-related column used by summary functions.

``Attachment``
   A column that refers to uploaded file attachments.

``UTM Zone``
   A UTM-zone column used when spatial geometry is created from
   coordinates.

.. TODO: Confirm that these names match the current labels in the
   administration interface. Explain how these semantic types relate to
   PostgreSQL data types and document any required PostgreSQL type for each
   option.

.. TODO: Clarify how latitude and longitude columns are paired and how the
   UTM zone, coordinate reference system, and hemisphere are determined
   during geometry creation.

.. _column-commands:

Column descriptions and commands
--------------------------------

The **Comment** field contains a description of the column's contents. Adding
a meaningful description is recommended because it contributes to the
project's metadata.

The **Command** field can be used to perform specific operations or assign
settings to a column. The documented commands include:

``SET srid:4326``
   Assign SRID 4326 to the ``obm_geometry`` column. Replace ``4326`` with
   the spatial reference identifier required by the project.

``SET use_rules:1``
   Enable access-rule handling for the ``obm_id`` column.

``RENAME:new_name``
   Rename a column to ``new_name``.

``DROP``
   Delete the column.

Renaming or deleting a column can invalidate upload forms, query templates,
modules, views, triggers, and external applications that refer to it. Update
all dependent configuration before performing either operation, and create a
database backup when appropriate.

.. TODO: Confirm the exact syntax, case sensitivity, and supported targets of
   every command. Explain whether commands are executed immediately and
   whether the interface checks database dependencies before renaming or
   deleting a column.

.. TODO: Clarify whether ``SET srid`` only changes metadata or transforms
   existing coordinates. Changing an SRID without transforming coordinate
   values can result in invalid spatial data.

.. TODO: Explain what ``SET use_rules:1`` changes and whether it creates,
   enables, or merely registers the project's row-level access rules.

.. _sql_console:

SQL console
-----------
An SQL console is also available to system administrators. The SQL console can 
be used to modify or delete project data and database structures. For this reason, 
access (to the database tables interface) should only be granted to trusted users 
who have sufficient experience with PostgreSQL and OpenBioMaps system administration 
tasks.

Queries executed in the SQL console can be saved and re-run.

The console displays the query results in a dynamic table. The results of the query table
can be exported as a CSV file. If the query results contain more than 1,000 rows, the 
table is no longer displayed; instead, a CSV export is automatically generated.

.. _sql_views:

Managing views
--------------

A data table can be replaced by a view to provide a customised
representation of its data or to improve a specific workflow. The documented
process creates a schema with the same name as the original table, moves the
original table into that schema, and creates a view in its previous
location. Corresponding ``INSERT``, ``UPDATE``, and ``DELETE`` rules provide
write operations where configured.

This approach may be useful for large tables affected by expensive
workflows or triggers. It changes the database structure substantially and
can affect forms, queries, modules, foreign keys, triggers, backups, and
external clients.

.. TODO: Document the exact transformation performed when a table is
   replaced by a view, including object names, ownership, privileges,
   sequences, indexes, constraints, foreign keys, and generated write rules.
   A supported rollback procedure should also be provided.

.. TODO: Explain which performance problems this feature is intended to
   solve. Replacing a table with a view does not by itself improve
   performance, so the expected view definition and optimisation strategy
   should be described.


.. _data-access-check:

Data access
===========

The **Data access** section summarises the project's access configuration and
the current state of row-level access rules. Administrators can inspect the
read and modification levels applied to the project and its managed data
tables.

The interface includes:

* the configured levels for reading and modifying data;
* the status of access restrictions for individual data tables;
* controls for enabling or disabling configured restrictions;
* the status of triggers used to maintain access rules; and
* links to related documentation.

The documented access levels are:

``everybody``
   Access is not restricted to authenticated users.

``logged-in users``
   Access requires authentication.

``specified group members``
   Access is controlled through project groups and more specific rules.

The effective access to a record may be affected by project-level,
row-level, and column-level rules. For a detailed overview, see
:doc:`Data access <../data_access>`.

The interface is available through **Profile > Project administration >
Data access**. Some underlying defaults may also be defined in the project's
``local_vars.php.inc`` configuration file.

.. TODO: Confirm the current navigation labels and map the interface labels
   ``everybody``, ``logged-in users``, and ``specified group members`` to the
   corresponding configuration values.

.. TODO: Document which changes can be made directly through this page and
   which still require editing ``local_vars.php.inc``. Explain how conflicts
   between interface settings and configuration-file values are resolved.


.. _group-settings:

Groups
======

The **Groups** section allows administrators to create and manage groups of
project users. Groups are used to assign access to data, upload forms,
modules, and administrative functions.

Administrators can:

* create a group;
* add users to or remove users from a group;
* add groups to other groups where nested groups are supported; and
* use the resulting groups in other access-management interfaces.

Nested groups can provide a reusable and scalable permission structure. They
should nevertheless be kept simple enough that administrators can determine
the effective permissions of an individual user.

.. TODO: Explain the exact behaviour of nested groups, including recursive
   membership, circular-reference prevention, and permission precedence.
   Document whether deleting a group removes its references from upload
   forms, access rules, modules, and administrative permissions.

.. TODO: Clarify whether group names can be changed after a group is used in
   access rules and whether access rules store a group identifier or its
   name.


.. _upload-forms:

Upload forms
============

Upload forms determine how data can be entered or imported into project
tables. They define the available fields, input controls, validation rules,
and access settings for a data-collection workflow.

For detailed instructions, see
:doc:`Upload form management <../upload_forms>`.


.. _trigger-functions:

Functions
=========

The **Functions** section provides tools for reviewing SQL rules and triggers
associated with project tables and views. It includes separate lists of the rules and
triggers registered for each table and provides templates for selected
trigger functions.

The interface can create, edit, enable, or disable the following documented
trigger types:

* taxon-list triggers;
* history triggers; and
* access-rules triggers.

Furthermore, custom triggers and rules can also be created and configured here.

Database triggers execute automatically when data change. An incorrect
trigger can reject valid changes, modify data unexpectedly, or weaken access
control. Test customised trigger functions before enabling them in a
production project.


Taxon-list trigger
------------------

The taxon-list trigger inserts previously unknown scientific names from a
configured species-name field into the project's taxon table. This can help
maintain a project whose species list expands as observations are added.

The species names added to the taxon table can now be maintained via the taxon name 
management interface.

:ref:`Administrative settings: species names <species-names>`


History trigger
---------------

The history trigger records changes made to records in the target table.
The resulting history can be displayed through the record's data-history
interface.

.. TODO: Document the operations and values recorded by the history trigger.
   Clarify whether it stores previous and new field values, timestamps,
   editor identities, transaction identifiers, or only a count of changes.
   Retention, access, restoration, and storage requirements should also be
   described.


Access-rules trigger
--------------------

The access-rules trigger maintains row-level access rules for records in a
project table. It can derive restrictions from a configured sensitivity
field and can transfer read and write permissions from the upload form used
to create a record.

For example, if an upload form grants read access to groups A and B and
write access to group C, the trigger can add those assignments to the
rules-table entry associated with each record created through that form.

This trigger is relevant to projects that use group-level or row-level
access restrictions. Its configuration must be consistent with the
project's general access settings and rules-table schema.

For more information, see :doc:`Data access <../data_access>`.

.. TODO: Explain how the trigger handles records created by SQL, the API, or
   another process that has no associated upload form. Document its
   behaviour when a record is updated, moved between uploads, or deleted.

.. TODO: Clarify whether enabling the trigger creates rules for existing
   records or only for subsequent changes. A supported procedure for
   regenerating and validating all rules should be documented.


.. _species-names:

Species names
=============

The **Species names** section manages the project's taxon table. Species
names can be assigned to the following documented categories:

* accepted name;
* synonym;
* common name; and
* misspelled name.

Names stored in the taxon table are used by taxon-related search interfaces
and by background jobs that detect or repair taxon names.

.. TODO: Confirm the current category names and correct the source-interface
   spelling of ``misspelled`` if necessary. Document the relationships
   allowed between accepted names, synonyms, common names, and misspelled
   variants.

.. TODO: Explain which taxonomic fields are stored, how names can be
   imported or exported, and how the interface prevents duplicate or
   circular synonym relationships.

.. TODO: Identify the current name and behaviour of the
   ``taxon-name-repair-background-jobs`` functionality and link to its
   configuration instructions.


.. _localisation:

Translations
============

OpenBioMaps uses global and project-specific translations.


Global translations
-------------------

Global translations can be added and improved through the
`OpenBioMaps translation platform
<https://translate.openbiomaps.org/>`_. The platform contains translations
for the web application, mobile applications, and other OpenBioMaps
components. Contributors can also propose a new language.

.. TODO: Document the account, review, and release workflow of the
   translation platform. Explain when an accepted global translation becomes
   available on an OpenBioMaps server.


Local translations
------------------

Local translations allow a project to define project-specific interface
text. Translation keys use the ``str_`` prefix followed by a descriptive
English identifier. For example, a project could define
``str_observations`` and provide its translation in each active language.

A public example is available at:

https://openbiomaps.org/projects/checkitout/upload/?form=426&type=web

.. TODO: Document where local translations are created, how active languages
   are selected, which components recognise local keys, and what happens
   when a translation is missing. Clarify whether local translations
   override global strings that use the same key.

.. TODO: Replace or supplement the public example with a stable screenshot
   or a description because the linked project and form identifier may
   change.


.. _module-settings:

Modules
=======

Modules extend the functionality available in an OpenBioMaps project. Their
configuration and access requirements depend on the individual module.

Modules extend the functionality available in an OpenBioMaps project. Their 
configuration and access requirements depend on the individual module.
Modules often provide basic functions, such as text search interfaces on the map 
page; in other cases, they provide tools specific to certain tasks. The behaviour 
of modules can often be customised.

For more information, see :doc:`Modules <../modules>`.


.. _interrupted-uploads:

Interrupted uploads
===================

The **Interrupted uploads** section lists saved or unfinished file uploads
and web-form data-entry sessions. Depending on their state, an interrupted
upload can be restored or discarded.

Administrators should verify that an upload is no longer required before
deleting it. An interrupted upload may contain work that its owner intends
to resume.

.. TODO: Document who can view, resume, or delete another user's interrupted
   upload. Explain the difference between a manually saved upload, an
   automatic backup, and an interrupted upload.

.. TODO: Specify retention periods, storage limits, automatic cleanup rules,
   and whether deleting an interrupted upload also deletes its uploaded
   temporary files.


.. _file-manager:

File manager
============

The **File manager** section provides tools for managing attachments uploaded
to the project. It can be used to browse attachments, review their
associations with database records, and create exports.

The documented functions include:

* listing uploaded attachments;
* filtering and sorting attachments;
* editing file comments;
* linking attachments to data records;
* managing existing file associations; and
* exporting the attachments associated with a data table.

A bulk export is processed as a background job. When processing has
finished, the system provides a link for downloading the resulting archive.

Access to attachment management and export functions should be limited to
authorised users. Exported files remain subject to the project's data-access
and privacy requirements.

.. TODO: Confirm the available filters and editable metadata. Document which
   attachment formats can be previewed and whether changing a file
   association also updates the corresponding record.

.. TODO: Explain how attachment exports apply row-level and column-level
   access rules, where generated archives are stored, how long download
   links remain valid, and who can use them.

.. TODO: Clarify whether deleting a file through this interface is supported
   and what happens to records, metadata, thumbnails, and backups that refer
   to the deleted file.


.. _sql-query-settings:

SQL query settings
==================

The **SQL query settings** section defines the templates used to assemble
queries for MapServer layers and for textual query results in the web
application. These templates resemble SQL but include OpenBioMaps
placeholders that are replaced dynamically by the query interpreter.

Each query template should be connected to a web-map layer. In a MapServer
mapfile, a WMS layer that uses a dynamically generated query must contain a
``DATA`` definition with the ``%query%`` placeholder.

Query templates can contain placeholders delimited by percent signs. Core
functions and installed modules may replace these placeholders with SQL
fragments at runtime.

.. TODO: Provide a complete reference for all supported placeholders,
   including their valid positions, replacement values, dependencies, and
   security constraints. The source text refers to both ``%morefilter%`` and
   ``%morefilters%``; confirm which form is valid.


Basic query template
--------------------

A query template may use placeholders such as ``%qstr%`` for query
conditions and ``%morefilter%`` for additional filters:

.. code-block:: sql

   SELECT obm_id, %grid_geometry% AS obm_geometry
       %selected%
   FROM %F%checkitout c%F%
       %uploading_join%
       %rules_join%
       %taxon_join%
       %grid_join%
       %search_join%
       %morefilter%
   WHERE %geometry_type% %envelope% %qstr%

The ``%F%`` markers identify the primary ``FROM`` relation and its alias so
that the interpreter can split and extend the template.

.. TODO: Explain why the primary relation must be enclosed in ``%F%``
   markers, whether schema-qualified and quoted names are supported, and
   which aliases are available to generated fragments.


Adding joins
------------

Additional joins can be enclosed in ``%J%`` markers:

.. code-block:: sql

   SELECT
       n.obm_geometry,
       n.obm_id,
       -2 AS date_part,
       nestbox_type,
       project_id,
       beinaction
       %selected%
   FROM %F%public_nestbox_data n%F%
       %J%LEFT JOIN public_nestbox_data_observations o
           ON o.nestbox_id = n.obm_id%J%
       %taxon_join%
       %morefilter%
   WHERE %envelope% %qstr%

.. TODO: Explain how multiple ``%J%`` blocks are processed and whether the
   interpreter can remove a join when none of the selected fields or filters
   require it.


Complex query templates
-----------------------

Templates can also use common table expressions and other SQL constructs:

.. code-block:: sql

   WITH aall AS (
       SELECT
           o.obm_id,
           n.obm_geometry,
           nestbox_type,
           project_id,
           beinaction,
           COALESCE(
               EXTRACT(DAY FROM (CURRENT_DATE - datum)::interval),
               '-1'
           ) AS date_part
           %selected%
       FROM %F%public_nestbox_data_observations o%F%
           %J%LEFT JOIN public_nestbox_data n
               ON nestbox_id = n.obm_id%J%
           %taxon_join%
           %morefilter%
       WHERE 1 = 1 %envelope% %qstr%
   )
   SELECT *
   FROM aall
   ORDER BY date_part DESC

A typical simple template has the following form:

.. code-block:: sql

   SELECT obm_id, obm_geometry %selected%
   FROM %F%checkitout c%F%
       %uploading_join%
       %rules_join%
       %taxon_join%
       %morefilter%
   WHERE %geometry_type% %envelope% %qstr%

Query templates affect both correctness and data access. An incorrect join
or missing access-rule placeholder may expose records or fields that should
be restricted. Test each query as public, authenticated, and group-specific
users before making it available.

.. TODO: Document which access-control placeholders are mandatory and
   whether the application rejects templates that omit them. Explain how
   parameter values are escaped or bound to prevent SQL injection.

.. TODO: Add a procedure for testing a template, inspecting the generated
   SQL, diagnosing placeholder errors, and restoring the previous version.


.. _map-settings:

Map settings
============

The **Map settings** section configures spatial layers in the web map and
their corresponding MapServer definitions. The web-map and MapServer
settings must remain consistent so that layers use the intended data source,
projection, extent, and style.


Web-map layers
--------------

The web-map settings configure the OpenLayers-based map interface.
Administrators can define settings such as:

* the initial map centre and zoom level;
* the available base maps and overlay layers;
* which layers are visible by default;
* the association between layers, project tables, and query templates; and
* selected aspects of layer appearance and behaviour.

.. TODO: Document every editable OpenLayers setting, its expected format,
   default value, and supported coordinate reference system. Explain how
   layer order, visibility ranges, opacity, legends, and queryability are
   configured.


MapServer settings
------------------

Advanced administrators can edit the project's raw MapServer mapfile. The
mapfile defines layer data sources, spatial reference systems, extents,
styles, and rendering options.

Changes to a mapfile can make project layers unavailable or expose an
unintended data source. Preserve a working version and validate the edited
mapfile before deploying changes.

.. TODO: Document where the mapfile is stored, whether edits are versioned,
   how its syntax can be validated, and how a previous configuration can be
   restored.

.. TODO: Explain which portions of the mapfile are generated by OpenBioMaps
   and which can be edited safely without being overwritten.


Spatial reference systems
-------------------------

Map layers must use correctly defined spatial reference systems. The
configured SRID determines how coordinates are interpreted and transformed
when data from different sources are displayed together.

The map extent and projection settings control the area and coordinate
system displayed by the web map. They must be compatible with the layer
data, MapServer configuration, and OpenLayers settings.

.. TODO: Identify the required projection for the web map, the supported
   source projections, and where transformations occur. Include guidance for
   choosing an extent and diagnosing layers displayed in the wrong location.


.. _member-settings:

Members
=======

The **Members** section lists the users registered in the project.
Administrators can manage project membership, status, and group assignments.

The documented member statuses are:

``Normal``
   The user receives the project's standard upload and query permissions.
   More specific group assignments and access rules may modify these
   permissions.

``Operator``
   The user has access to all project functions and data.

``Suspended``
   The user cannot access project functions or data. Suspending a user is
   similar to disabling their project membership but does not delete their
   profile.

The project founder has full project access and does not need to be assigned
the operator status. Group assignments can be changed on this page, although
the **Groups** interface may be more convenient for managing several users.

For related settings, see :ref:`Groups <groups>` and
:ref:`Administrative access <administrative-access>`.

.. TODO: Confirm the current status names and define the exact permissions of
   founders, owners, hosts, operators, and normal users. These role names
   should be harmonised across the documentation.

.. TODO: Explain whether suspension affects only the current project or the
   user's server-wide account. Document its effect on API tokens, active
   sessions, scheduled jobs, record ownership, invitations, and messages.


Viewing another user's profile
------------------------------

A member's name links to their profile page. Administrators with the
required permission may see a user-secret icon in the upper-right area of
the page. This function opens another user's profile while the administrator
remains authenticated with their own account.

The icon used by the interface is documented by
`Fork Awesome
<https://forkaweso.me/Fork-Awesome/icon/user-secret/>`_.

This feature can expose personal information and user-specific content.
Access should be restricted and its use should follow the project's privacy
and auditing policies.

.. TODO: Clarify whether this function impersonates the user or only allows
   an administrative view of the profile. Document which actions are
   permitted, whether the affected user is notified, and whether access is
   recorded in an audit log.


.. _message-templates:

Message templates
=================

The message template editor is currently unavailable.

Messages sent automatically by the system or a project are generated from
templates. OpenBioMaps provides global templates for implemented message
types, and a project can create local versions that override them.

To customise a global template, select it, edit its contents, and save it as
a local version. Templates may contain variables that are replaced when the
message is sent. The variables supported by an individual template are
defined by the function, module, or background job that sends it.

New templates can also be created for custom modules and background jobs.

.. TODO: Document the template fields, supported message formats, language
   handling, fallback order, and procedure for reverting a local override to
   the global version.

.. TODO: Explain whether template content is escaped and which HTML or
   markup is permitted. Template editing should be assessed for risks such
   as unsafe links, HTML injection, and unintended disclosure of variables.


Variables and included templates
--------------------------------

Variables are written between percent signs, for example ``%USER_NAME%``.
The following global variables are documented:

``%PROJECT_TABLE%``
   The project's database identifier or table name.

``%PROJECT_TITLE%``
   The project's short description.

``%PROJECT_DESCRIPTION%``
   The project's long description.

``%USER_NAME%``
   The name of the recipient or relevant user.

``%URL%``
   A URL associated with the message.

``%OB_DOMAIN%``
   The OpenBioMaps domain associated with the message.

``%DOMAIN%``
   The domain name defined in the ``projects`` table.

``%PROTOCOL%``
   The protocol defined in the ``projects`` table.

One template can include another template. For example, appending
``@footer@`` includes the template named ``footer``.

.. TODO: Confirm the exact meaning and availability of each global variable.
   In particular, distinguish ``%PROJECT_TABLE%``, ``%OB_DOMAIN%``,
   ``%DOMAIN%``, and ``%URL%``.

.. TODO: Document whether included templates can themselves include other
   templates, how missing variables or templates are handled, and whether
   recursive inclusion is prevented.


Predefined templates
--------------------

The documented user-related templates are:

``welcome_to``
   Welcomes a user to the project.

``change_email_address``
   Sends a confirmation link for changing a user's email address.

``dropmyaccount``
   Confirms a request to delete an account.

``create_new_project``
   Confirms the creation of a project.

``invitation``
   Sends an invitation to join a project.

``invitation_accomplished``
   Reports that an invitation has been accepted.

``invitation_request``
   Notifies administrators about a request for an invitation.

``lostpw``
   Supports password recovery.

The documented general-purpose templates are:

``new_gitlab_issue``
   Contains a copy of a submitted bug report.

``new_shared_polygon``
   Announces a newly shared polygon.

``new_upload_news``
   Announces a new upload in the project news.

``new_upload_report``
   Notifies administrators about a new upload.

``footer``
   Provides a general message footer.

``interconnect_request``
   Supports an interconnection request.

The documented evaluation-notification templates are:

``data_evaluation_commenters``
   Notifies previous commenters when a record receives a new comment.

``data_evaluation_owner``
   Notifies the owner when a record they uploaded receives a comment.

``upload_evaluation_commenters``
   Notifies previous commenters when an upload receives a new comment.

``upload_evaluation_owner``
   Notifies the owner when their upload receives a comment.

``user_evaluation_commenters``
   Notifies previous commenters when a user receives a new comment.

``user_evaluation_owner``
   Notifies a user when they receive a comment.

The documented module-related templates are:

``dlr_new_request``
   Notifies project administrators about a new download request. The
   documented variables are ``username``, ``requestid``, and
   ``request_message``.

``dlr_request_registered``
   Confirms to a user that their download request has been registered.

``incomplete_list_processed``
   Reports that an incomplete list has been processed.

``incomplete_list_unprocessed``
   Reports that an incomplete list could not be processed.

.. TODO: Verify that all template identifiers are current and add the
   variables available to each template. The purposes of
   ``interconnect_request``, ``incomplete_list_processed``, and
   ``incomplete_list_unprocessed`` require further explanation.

.. TODO: Clarify whether ``dropmyaccount`` deletes a server-wide account or
   only project membership, and whether ``create_new_project`` is a
   confirmation request or a notification after creation.


.. _server-info:

Server info
===========

The **Server info** section displays selected information about the
OpenBioMaps server and the resources used by the project. Depending on the
server configuration, it may include:

* the installed OpenBioMaps application version;
* disk usage by project files, attachments, and uploads;
* load averages for the previous 1, 5, and 15 minutes;
* server load normalised by the number of CPU cores;
* available memory; and
* a link to the Supervisor administration interface.

These values can help administrators identify resource constraints and
provide diagnostic information to server operators. Access to detailed
server information should be restricted because version and infrastructure
details may be security-sensitive.

.. TODO: Confirm which values are available to project administrators and
   which require server-level privileges. Document the units, update
   interval, data source, warning thresholds, and interpretation of each
   metric.

.. TODO: Clarify whether the Supervisor link is always available, which
   Supervisor product it refers to, and how access to that external
   interface is authenticated.


.. _server-logs:

Server logs
===========

The **Server logs** section provides access to logs made available by the
server configuration. The documented sources include:

* application or system logs;
* MapServer logs;
* background-job events; and
* background-job errors.

The interface may provide filtering, searching. Logs can
contain usernames, record identifiers, query details, file paths, request
parameters, or other sensitive information. Access and retention should
follow the server's security and privacy policies.

.. TODO: Confirm the available log sources and whether live updates are
   currently supported. Document the location, format, time zone, rotation,
   retention, and maximum result size of each log.

.. TODO: Explain which personal or confidential data can appear in logs and
   how administrators can download, redact, or delete log content. It should
   also be stated whether viewing logs is itself audited.


.. _background-job-settings:

Background job settings
=======================

Background jobs allow a project to execute scheduled or manually initiated
tasks without continuous user interaction. They can be used for operations
such as:

* maintaining species-name data;
* validating records;
* importing or exporting data;
* cleaning temporary tables;
* running analyses; and
* refreshing materialised views.

A background job is a standalone program. OpenBioMaps jobs are commonly
written in PHP, but the server may also support programs written in Python,
R, Bash, or another installed language.

The administration interface can be used to:

* install predefined jobs from a central Git repository;
* upload a project-specific job;
* review installed jobs;
* configure job parameters and schedules;
* enable or disable jobs;
* start a job manually;
* inspect recent output and execution status; and
* edit job source code where this function is enabled.

Detailed logs are available through the **Server logs** section.

Editing or uploading a job is equivalent to installing executable code on
the server. These functions must be restricted to trusted administrators,
and custom jobs should be reviewed for command injection, unsafe file
access, credential exposure, and excessive resource use.

.. TODO: Document the required package structure, manifest, entry point,
   supported languages, execution user, working directory, environment
   variables, dependencies, timeout, and resource limits of a job.

.. TODO: Explain how jobs from the central repository are authenticated,
   versioned, updated, and reviewed. Clarify whether local changes are
   overwritten by an update and how a previous version can be restored.


For more information, see :doc:`Jobs <../jobs>`.

Scheduling jobs
---------------

The server's system-level scheduler must first be configured. In a Docker
installation, this is typically a cron process on the host. It periodically
invokes the scheduler for the project, which starts any jobs that are due.

Before scheduling a newly installed or modified job:

#. review its configuration and source;
#. use **Run** to execute it manually;
#. wait for the execution to finish;
#. inspect its result and logs; and
#. configure the recurring schedule only after the test succeeds.

The project scheduler uses cron-like minute, hour, and day fields. An
asterisk means every valid value in the corresponding field.

.. TODO: Document all scheduler fields and their accepted cron syntax,
   including ranges, lists, steps, month, weekday, and time zone. Clarify
   whether overlapping executions of the same job are prevented.


System-level Docker example
---------------------------

The following example invokes a project's scheduler from the host:

.. code-block:: console

   */5 * * * * /usr/local/bin/docker-compose -f /srv/docker/openbiomaps/docker-compose.yml exec -u www-data -T app php /var/www/html/biomaps/root-site/projects/myproject/jobs.php

Replace the Compose file, service, project path, and execution user with
values appropriate for the installation.

.. TODO: Verify this command against the currently supported Docker
   installation. Newer installations may use ``docker compose`` rather than
   ``docker-compose``.

.. TODO: State the recommended invocation interval and explain whether
   running this command every five minutes prevents jobs with one-minute
   schedules from running at their intended time. Include logging and
   failure-notification recommendations for the host-level cron task.


.. _project-description:

Project description
===================

The **Project description** section defines the project name displayed in
the page header and the longer project description. Separate values can be
provided for each active language.

The short and long descriptions may also be used in project metadata,
message templates, and summary pages. They should therefore identify the
project clearly and provide current contact or contextual information where
appropriate.

.. TODO: Document the supported formatting, maximum lengths, fallback
   language, and every interface in which the short and long descriptions
   appear. Clarify whether these values correspond exactly to
   ``%PROJECT_TITLE%`` and ``%PROJECT_DESCRIPTION%`` in message templates.


.. _data-management-page:

Data management
===============

The **Data management** section provides summaries of uploads and
observation lists. It can help administrators review recent submissions,
identify contributors, and navigate between related records, uploads, and
tracklogs.

The documented functions include:

* listing observation lists by uploader, date, or tracklog;
* summarising the number of records uploaded by each user and to each table;
* displaying observation lists submitted during the previous 90 days; and
* displaying tracklogs submitted during the previous 30 days.

Interactive tables provide filtering and sorting where supported.

.. TODO: Define an ``observation list`` and explain how it relates to an
   upload, individual records, and a tracklog. Document the links and actions
   available from each summary.

.. TODO: Confirm whether the 90-day and 30-day intervals are fixed,
   configurable, or merely defaults. Explain which time zone and timestamp
   are used to select recent activity.

.. TODO: Clarify whether summaries include deleted, rejected, or partially
   completed uploads and how row-level access restrictions affect the values
   shown to an administrator.
