# Modules

Modules are configurable extensions to the OpenBioMaps web application. They
can add user-interface components, data-processing functions, export formats,
administrative tools, APIs, or integrations with external services.

There are two main module scopes:

- **Project-level modules** provide functions that apply to the whole project,
  such as spatial-shape management, attachment support, or PostgreSQL user
  creation.
- **Table-level modules** apply to a particular data table, such as map-page
  filters, result displays, data transformations, or export formats.

Modules are connected to hooks in the application. Most user-facing hooks are
located on the map page and the profile page, although modules can also add
administration pages, APIs, background jobs, and upload-related functions.

Most modules accept parameters in JSON format. Some modules instead provide a
dedicated administration interface, and some require both JSON parameters and
additional database or MapServer configuration.

> **Version compatibility:** The available modules and their parameters can
> change between OpenBioMaps releases. The module administration page of the
> installed application is the authoritative list of modules available to a
> project. Check the module source and release notes before copying a
> configuration from another installation.

## Module administration

Modules can be enabled and configured on the **Project administration →
Modules** page.

A module can usually be:

- added to the project;
- assigned to users or groups;
- configured with JSON parameters;
- enabled or disabled; and
- opened through a module-specific administration page, if one is provided.

Module names and JSON keys are case-sensitive. Validate JSON before saving it.
JSON does not allow comments or trailing commas.

### Adding a custom module

Custom modules can be uploaded and added to a project. Developers should use
the example modules in `resources/includes/modules/examples/` as a starting
point and compare their implementation with modules included in the installed
OpenBioMaps release.

Custom module code must be reviewed before deployment. A module runs as part
of the application and may have access to project data, the authenticated
user's session, and database connections.

#### Module class contract

Modules extend the `module` base class. The application constructs a module as
`new your_module($method, $stored_params, $runtime_arguments)`. The inherited
constructor normalizes the stored parameters into `$this->params`, checks
access, and calls the requested method. A module should keep the inherited
constructor instead of defining its own, so that these checks always run.

- Automatic hooks run only if their names are listed in the `Methods` list of
  the module metadata. Hook discovery does not use PHP reflection.
- `Module-type` is `table`, `project`, or `both`. Choose the narrowest scope
  that applies.
- Methods listed in `$guarded_methods` require a signed-in user. Methods listed
  in `$master_access_only` require master access; `adminPage` is master-only by
  default.
- `$ajax_allowed_actions` restricts the inherited AJAX dispatch to the listed
  public actions, and `$ajax_deps` lists the includes loaded for it.
- The optional protected methods `prepare_module()` and `finish_module()` run
  before and after a successful dispatch. `prepare_module()` can cancel the
  call by returning `false`.
- `init()` runs after registration and after an enabled registration is
  updated, so it must be idempotent. `uninstall()` (or the inherited
  `destroy()`) runs before disabling and can abort it by returning
  `['error' => 'message']`. Call the parent methods to enforce declared
  dependencies.
- System modules are installed in `resources/includes/modules/`; project-local
  modules and overrides are installed in `resources/local/includes/modules/`.

The commented reference implementation is
`resources/includes/modules/examples/official_module_template.php`.

### Module access

The same module can be added more than once with different access settings or
parameters. This allows administrators to provide different configurations
to different users, groups, or tables.

For example:

- `allowed_columns` can expose different columns to different groups; and
- `text_filter` can provide table-specific filter columns in a project that
  contains multiple data tables.

The **Access** column controls the general audience of a module instance. The
available choices include public access and access restricted to logged-in
users.

The **Group access** column further restricts the module instance to selected
project groups or individual users.

When several instances of the same module apply to one user, test which
configuration is selected or combined by the installed OpenBioMaps version.
Avoid overlapping access rules unless their behaviour is understood.

### Enabling and disabling modules

Each configured module instance can be enabled or disabled. Disabling a
module preserves its configuration but prevents it from being used.

After changing a module's status, test the relevant page with users from each
affected access group. Some modules also create database objects or retain
module-specific settings when disabled.

### Module dependencies

A module can declare other modules as dependencies. Required modules must be
enabled before the dependent module can be enabled. Conversely, a required
module cannot be disabled while an enabled module depends on it.

The application checks these rules in the modules' installation and removal
lifecycle hooks. If a check fails, the requested state change is rejected and
the module administration page displays the error. Disable dependent modules
before disabling their requirements.

Module authors declare dependencies with the `Dependencies` list in the module
metadata. The dependency must be available and enabled in the scope required by
the dependent module.

### Removing modules

The module administration interface does not currently provide a general
option for removing an installed module from the application.

A configured module instance may be disabled. Do not manually delete module
files or database objects unless the module's removal procedure is known and
a backup is available.

### Module parameters

Most modules accept JSON parameters directly on the module administration
page. Other modules provide a dedicated administration tab for module-specific
tasks. `geometry_management` is an example of a module with its own
administrative interface.

Examples in this document use placeholders such as `YOURTABLE`, `column_name`,
and `schema.table`. Replace these placeholders with identifiers from the
project.

## Project-level modules

.. _protocol-installer-guide:

### `protocol_installer`

The `protocol_installer` module lets project masters install collection
protocols from a configured public catalogue. Its administration page guides
table and column mapping, list configuration, plan review, installation, and
supported upgrades.

**Parameters:** None. The catalogue is configured by the server administrator.

The installer reads declarative YAML from one configured public GitLab repository.
The :doc:`package format <protocol_package_format>`
is the normative version 1 contract. Published protocols are maintained in the
[share-your-protocol repository](https://gitlab.com/openbiomaps/share-your-protocol).

#### Configuring the protocol source

Set these server constants in `/etc/openbiomaps/system_vars.php.inc`:

```php
define('PROTOCOL_INSTALLER_GITLAB_PROJECT', 'openbiomaps/share-your-protocol');
define('PROTOCOL_INSTALLER_GITLAB_REF', 'main');
define('PROTOCOL_INSTALLER_GITLAB_CATALOGUE_PATH', 'catalog.yml');
```

The project value is a GitLab `group/project` path, not a URL. The ref and
catalogue path are optional and default to `main` and `catalog.yml`. The
repository must be public; this release does not authenticate to GitLab.
Enable `protocol_installer` for the project through module administration.

#### Installing a protocol

1. Open **Data collection protocols** as a project master. The catalogue shows
   whether each protocol is installed, its installed version, and whether a
   newer version is available. Choose **Install** or **Upgrade to** the latest
   version, then inspect its requirements. Earlier catalogue versions remain
   available under **Other catalogue versions**.
2. Map every logical table to an accessible project table or create a new one.
   A `project_main` table always maps to the registered main table. For each
   column choose a compatible existing column or create it. Table creation and
   upload-form permissions are required when the plan uses those operations.
3. Configure lists and form names. Choose an embedded or terms-backed fixed
   list and configure each project-specific list via `list_definition` JSON
   or an existing terms source. Check web and API form settings and access values.
4. Review all operations, conflicts, and warnings. Resolve every blocking
   conflict and rebuild the plan after a mapping change. Confirm installation
   explicitly. The result links to installed forms and shows the saved mapping.

#### Upgrading a protocol

The same protocol can be installed once per project. A repeated submission
returns the existing installation. To upgrade, select a newer version of the
same protocol. The wizard shows the installed version, locks existing mappings
and local settings, and plans only new objects. Review conflicts and operations
before confirming. Existing forms, columns, lists, and records remain in place.

An installed form can gain fields when its earlier fields and other definition
properties remain unchanged. The new fields are appended after existing local
fields, preserving the form identity, access settings, and current field order.
Changing or removing an earlier field, or changing other existing package
elements, still blocks the automatic upgrade.

#### Completing an upgrade manually

For a conflicting upgrade, the plan lists each manual task and shows the old
and target definitions where available. Complete the changes in the project,
check data and forms, map manually created tables and columns to their existing
destinations, and rebuild the plan. A project master can then check the manual
completion declaration and select **Mark upgrade complete**. The server checks
the reviewed plan again, requires target tables and columns to exist, checks
new forms and imported terms, and rejects pending create operations or unrelated
blocking conflicts.

The declaration records the administrator, time, versions,
and acknowledged conflicts in `manual_upgrade_history`; it advances the saved
package version and mappings without claiming manually created objects as
installer-owned. Definition changes that cannot be verified from inventory
remain the administrator's responsibility.

#### Resuming an upgrade

If an upgrade stops, select that protocol and choose **Resume recorded upgrade**.
Recovery uses the saved package and plan, even if a different catalogue revision
is selected later. Finish a recorded upgrade before starting another one or
uninstalling the protocol in development mode.

#### Protocol compatibility and safety

Existing columns must have compatible storage type and capacity. The planner
does not convert existing data or change existing column definitions. It checks
managed system columns, geometry requirements, field types, list sources,
metadata, and form-name collisions. An unrelated form or metadata record is
never overwritten. New physical names cannot collide with existing relations.

For a bounded existing `varchar(n)` column, declare `data_type: varchar` with
`max_length` no greater than `n` in the package. An unbounded `text` package
column cannot be mapped to bounded `varchar(n)` because future values could
exceed the destination's limit.

For terms import, a code with a different label or owner is a conflict; shared
terms are not silently changed. The installer validates YAML and references
before planning, then checks the current inventory again before execution.

#### Installation journal and recovery

The GIS and metadata databases commit separately. The installation record in
`public.protocol_installations` stores the protocol identity, version, GitLab
revision, checksum, package snapshot, mappings, plan, phase statuses, created
objects, and any error. Forms become active only after metadata completion.

If installation stops, inspect `status`, `gis_status`, `metadata_status`, and
`error` for the protocol and project. Reopen the same package in the wizard and
choose **Resume recorded installation**. Recovery uses the saved plan and only
installation-owned objects. Keep the original package revision available until
the installation is complete; do not manually remove partially created objects.
If the package identity or version differs, select the recorded version.

#### Uninstalling a protocol and version 1 limitations

Development mode offers a preview and confirmed uninstall for a recorded
installation. It removes its owned tables, columns, forms, imported terms,
journal entry, and GIS ownership markers. Reused project objects remain.
Dependencies from other forms block removal. Production uninstall and automatic
non-additive protocol version upgrades are deferred. Generic data imports,
package hooks, dependencies, and standardized exports are outside version 1.

#### Authoring and publishing a protocol package

1. Give the protocol a stable reverse-DNS `protocol_id` and a
   `protocol_version`. Keep logical table, column, form, and list IDs stable.
   Change suggested physical names only as hints; references must use logical
   IDs. Published updates must retain existing definitions and add new elements
   with new logical IDs. Existing forms and list definitions are preserved.
2. Declare each table as `project_main` or `create`, its portable columns,
   lists, and web/API forms. Use `system.geometry` and `system.files` for the
   application's managed fields. Use only the types and settings listed in the
   :doc:`format contract <protocol_package_format>`.
3. Choose list ownership deliberately. A `fixed` list can stay embedded or be
   imported into the project terms table. Its `code` is stored and its `label`
   is shown in both web and API forms. A `project` list is configured locally
   during installation. A `table` list refers to columns of a declared table.
   The package must not contain PHP, SQL, credentials, or physical project IDs.
4. Add the YAML path to `catalog.yml` in the same GitLab repository. The
   catalogue entry's ID and version must match the package. Paths are relative
   to the repository root; they cannot leave it. Commit both files and test
   that the selected branch contains them. The installer pins that commit and
   records the package checksum in its installation journal.

#### Developer regression checks

Before database tests, run `tools/sync-sablon-from-resources.sh --apply` in the
web-app repository to refresh the test fixture; do not edit `projects/sablon`
manually. Run the focused package, planning, inventory, and execution PHPUnit
tests in the Docker test environment. The browser smoke suite checks public
and project routes. For a full pilot, install protocols into a new dedicated
project and a project containing
existing records, then open a web form and retrieve an API form through the
mobile client. Verify that a fixed list stores its code with both embedded
and terms-backed choices.

### `classification`

The `classification` module manages controlled vocabularies (classification
schemes) for a project. A scheme contains concepts with multilingual labels and
stable identifiers (UUIDs). Schemes can be flat or hierarchical. A concept can
belong to several schemes, can have broader (parent) concepts within a scheme,
and can be linked to related concepts. Concepts and memberships can be
archived and restored, and unused concepts can be deleted.

The module has its own installation and administration page and does not
depend on `geometry_management`. Enabling the module installs or validates the
classification schema in the GIS database (recorded in
`system.classification_installation`) without recreating or reclassifying
existing data. Initialization is idempotent: if it fails, correct the problem
and enable the module again. The installation rejects an incompatible existing
schema instead of changing it.

Vocabulary administration is restricted to project masters. The administration
page provides:

- creating and editing schemes;
- creating concepts, editing their labels, and adding an existing concept to
  another scheme;
- adding and removing broader and related concepts;
- archiving, restoring, and deleting unused concepts and memberships; and
- searching concepts and paging through the results.

Disabling the module is blocked while an active or published web, file, or API
form contains a classification field; the error message lists the affected
forms. Disabling preserves all classification data and re-enabling makes it
available again. There is no automatic removal script, and rolling back to an
older version requires a reviewed backup.

**Parameters:** None.

**Release status:** this release provides vocabulary management. Assigning
classifications to sampling units and observations, vocabulary CSV import,
classification fields in upload forms, and the text filters are planned and not
yet available.

### `geometry_management`

The `geometry_management` module manages reusable spatial shapes. Users can
upload points, lines, and polygons, share them with other users, and use them
as record geometries during web or file uploads. Available upload geometries
can also be downloaded and displayed by the mobile application.

Newly uploaded shapes are not visible to other users by default. Project
administrators can grant permission to use each shape for queries or data
uploads. Users manage shapes on the **Shared geometries** page of their profile.
It has a **My geometries** tab and a **Community public geometries** tab, a
search box, filters for the use (query or upload) and the geometry type,
GeoJSON download, selection based deletion, and a form for uploading new
spatial files. Project administrators see all geometries of the project on the
**All polygons** administration tab and can assign each shape to selected
project users.

The module works independently of the `classification` module.

If an upload form uses an `obm_geometry` field, its map control can offer a
**Geometry from list** option. Selecting a named shape inserts its WKT
geometry into the upload field.

**Parameters:** None. The module uses its dedicated administration interface.

### `photos`

The `photos` module enables photo and other attachment fields on upload forms
and displays attached images on record data-sheet pages.

File-size limits, permitted file types, storage, access control, and backup
requirements must also be configured at the application and server levels.

**Parameters:** None.

### `create_pg_user`

The `create_pg_user` module allows authorised users to create personal
PostgreSQL accounts.

When the module is enabled:

- a **Create PostgreSQL user** box appears on the profile page for authorised
  users;
- users can create and renew their own database account;
- the generated account is assigned to the project's PostgreSQL user group;
  and
- the account can be used by database clients such as QGIS.

By default, the generated account:

- has read access to the project's database tables;
- is limited to one simultaneous client connection; and
- expires after one year.

The generated account is added to the PostgreSQL group named after the
project, commonly in the form `PROJECT_user`. A database administrator can
grant additional permissions, such as write access to selected tables, but
should follow the principle of least privilege.

Users can renew their access before or after expiry, subject to the installed
module's rules.

The following screenshot shows an example PostgreSQL/PostGIS connection in
QGIS:

![Adding an OpenBioMaps PostGIS connection in QGIS](images/qgis_connect.jpg)

Do not expose PostgreSQL to the public internet without appropriate firewall,
TLS, authentication, and access-control settings.

**Parameters:** None. Current releases may provide a dedicated administration
page.

### `computation`

The `computation` module provides project-specific computation functions.

Its exact behaviour depends on the installed module version and project
configuration. Review the module implementation before enabling it in a
production project.

**Parameters:** None documented.

### `custom_filetype`

The `custom_filetype` module supports project-specific preparation of custom
download formats, such as an Observado-style CSV file.

The output format and any required custom implementation depend on the
project.

**Parameters:** None documented.

### `taxon_meta`

The `taxon_meta` module provides taxon-related metadata functions.

Its user interface, required database structure, and configuration should be
verified against the installed module version.

**Parameters:** None documented.

### `turnstile`

The `turnstile` module integrates Cloudflare Turnstile into the registration
and invitation forms. When it is configured, the registration form shows the
Turnstile widget instead of the legacy image captcha, and the server verifies
the submitted token against the Turnstile API. The legacy captcha remains the
fallback when the module is not configured.

The module is configured with constants in the project configuration
(`local_vars.php.inc`), not with module parameters:

```php
define('CLOUDFLARE_TURNSTILE_SITE_KEY', 'your-site-key');
define('CLOUDFLARE_TURNSTILE_SECRET_KEY', 'your-secret-key');
```

The module is enabled only if `CLOUDFLARE_TURNSTILE_SECRET_KEY` is not empty.
Always define both constants: if the site key is missing, the module falls
back to using the secret key as the site key. The widget script is loaded from
`https://challenges.cloudflare.com/turnstile/v0/api.js`, and the verification
request is sent from the server to Cloudflare, so the server needs outbound
HTTPS access.

**Parameters:** None.

### `cms`

The `cms` module adds a **Content** group with **Pages** and **Posts** entries
to the profile menu. It is intended to connect the project dashboard with the
Nextcloud API. The administration page is available to project masters only.

The module is under development (version 0.0.1) and its administration page
does not provide settings yet.

**Parameters:** None.

### `ioc_bird_list`

The `ioc_bird_list` module imports the IOC World Bird List into the shared
database schema. The project administration page has an update button that
downloads the master list XML from `www.worldbirdnames.org`, creates a table
named `shared.ioc_<version>` (dots in the version are replaced by underscores)
and fills it with the list. The existing `shared.ioc_*` tables are listed on
the administration page.

The administration page is available to project masters only. The server needs
outbound HTTP access to download the list.

**Parameters:** None.

### `natura2000`

The `natura2000` module provides a Natura 2000 validation interface in the user
profile. It compares the observations of the project with the species lists
and sites stored in the `natura2000` database schema (for example the
`natura2000.sites`, `natura2000.mdb_species` and `natura2000.natura2000_fajok`
tables), and it includes a page for managing the related documents.

The module depends on this schema and on Hungarian site and species data,
therefore it is useful only in projects prepared for it. Its interface and
messages are currently Hungarian only. The profile page loads the Tabulator
library from `unpkg.com`.

**Parameters:** None documented.

### `pds_test_project_installer`

The `pds_test_project_installer` module is a transition entry for projects that
have the legacy PDS test installer enabled. It no longer installs or changes
project data. New PDS test catalogues are installed with the
`protocol_installer` module by selecting the PDS mobile form test catalogue.
Disabling the module leaves existing PDS installations untouched.

After installing the catalogue, enable `pds_test_evaluator` in module
administration.

**Parameters:** None.

## Table-level modules

### `box_load_selection`

The `box_load_selection` module uses reusable geometries for spatial queries
on the map page. It can be enabled, configured, and access-restricted
independently for each project table. It depends on the project-level
`geometry_management` module; enable `geometry_management` first.

When enabled for the current table, a **Spatial query** box appears on the map
page. Users can select an available project geometry and run a spatial query
against it. For polygon geometries, the interface may allow users to choose
whether records intersecting the polygon boundary are included.

**Parameters:** An optional JSON list can restrict the available spatial
relationships. If omitted, the module offers `contains`, `intersects`,
`crosses`, and `disjoint` where applicable.

### `additional_columns`

The `additional_columns` module defines columns used to associate records
across multiple data tables.

When tables are connected through a shared identifier, queries can include
the related records for that identifier. Users can bypass these joins by
selecting **Ignore table joins** on the map page.

For example, a project may store parent and offspring records in separate
tables and use a shared burrow identifier as the join column.

Use this module together with `join_tables`.

The module returns:

- an array of columns at index `0`; and
- an associative array of column names at index `1`.

**Parameters:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

### `allowed_columns`

The `allowed_columns` module supplements row-level data-access rules with
column-level restrictions.

Row-level rules determine which records a user can access. This module
determines which columns remain visible when a record is subject to a
`restricted` or `no-geom` rule, or when no matching rule exists.

The module is intended for projects whose base access level is not public and
whose data tables use the corresponding rules table.

**Parameters:**

```json
{
  "for_sensitive_data": [
    "column_visible_for_sensitive_records"
  ],
  "for_no-geom_data": [
    "column_visible_for_records_without_geometry_access"
  ],
  "for_general": [
    "column_visible_when_no_rule_matches"
  ]
}
```

Parameter meanings:

- `for_sensitive_data` lists columns visible for sensitive records.
- `for_no-geom_data` lists columns visible for `no-geom` records. If this key
  is omitted, all columns are accessible for those records.
- `for_general` lists columns visible when no rule matches. If this key is
  omitted, all columns are restricted in that case.

Test the effective permissions with public, authenticated, group-member, and
administrator accounts. Column restrictions must not be treated as a
replacement for correct database and API access control.

### `bold_yellow`

The `bold_yellow` module identifies important fields in result summaries.

Configured columns are highlighted in bold yellow in detailed result lists.
The mobile application also uses this configuration to select values shown in
the **Collected data** summary labels.

**Parameters:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

### `box_load_coord`

The `box_load_coord` module adds a **Position** block below the map.

The block:

- displays the coordinates of the current pointer position; and
- allows a user to enter latitude and longitude values and place the
  corresponding point on the map.

The parameters map user-facing coordinate-system names to EPSG codes.

**Parameters:**

```json
{
  "wgs84": "4326",
  "eov": "23700"
}
```

Only configure coordinate systems supported by the project and its mapping
components.

### `box_load_last_data`

The `box_load_last_data` module adds a **Quick queries** box to the map page.

It provides queries for:

- the current user's most recent upload;
- the most recent upload by any user; and
- the most recently uploaded records.

The first two options return one record. The parameter controls the number of
records returned by the third option. The documented default is 10.

**Parameters:**

```json
[
  10
]
```

### `box_custom`

The `box_custom` module loads a project-specific custom box on the map page.

The custom implementation must be placed in the project's
`local/includes/modules/` directory. Its class must provide at least the
`print_box()` and `print_js()` methods.

For a custom module stored in:

`local/includes/modules/hrsz_query.php`

the parameter contains the file's base name:

```json
[
  "hrsz_query"
]
```

The corresponding class is expected to be named `hrsz_query_Class`.

Custom module code must validate input, escape output, enforce permissions,
and use parameterised database queries.

### `identify_point`

The `identify_point` module allows users to identify one or more points on the
map and displays selected attribute values in a map popup.

**Parameters:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

Only include columns that the module's intended audience is permitted to
access.

### `cameratrap_api`

The `cameratrap_api` module provides communication between a camera-trap
dashboard and the Nextcloud API.

Its functions include:

- managing cameras and analyses;
- uploading and downloading images;
- starting analyses; and
- managing the Nextcloud credentials required by the integration.

The module creates or uses module-specific database objects. Review its SQL
installation file and access requirements before enabling it.

**Parameters:** None documented.

### `nextcloud_connect`

The `nextcloud_connect` module connects OpenBioMaps to a Nextcloud server. It
provides user-profile integration and issues JWT tokens for authentication.

Nextcloud URLs, credentials, signing secrets, token lifetimes, and TLS
validation must be configured securely through the mechanisms expected by the
installed release.

**Parameters:** None documented.

### `validation`

The `validation` module provides an internal API and administration interface
for data-validation algorithms.

Its functions include:

- managing validation rules;
- validating records; and
- logging validation actions.

Project-specific validation implementations may perform additional checks on
uploaded data.

**Parameters:** None documented. Additional rules are managed through the
module's administration interface and validation components.

### `download_restricted`

The `download_restricted` module introduces an administrator-controlled
download-authorisation workflow.

Instead of receiving immediate access to a download, users submit a request
describing the intended use of the data. Administrators can approve or reject
the request through the module administration interface.

The module provides:

- a download-request form;
- an administrator approval workflow; and
- integration with `results_buttons`.

When it is used with `results_buttons`, export options are made available only
to users whose request and permissions allow the download.

Enabling this module does not remove the need for server-side access checks.
Test direct export URLs and APIs to verify that download restrictions cannot
be bypassed.

**Parameters:** None. The module uses its dedicated administration interface.

### `extra_params`

The `extra_params` extension supplies additional input parameters to forms.

The exact syntax and availability of this extension must be checked against
the installed OpenBioMaps release because a standalone module with this name
may not be present in every version.

**Parameters:** No stable parameter format is documented here.

### `grid_view`

The `grid_view` module displays data using alternative polygon grids. Examples
include UTM grids, KEF grids, snapped points, and dynamically generated grid
polygons.

When a grid view is active, the geometry supplied by the module is used in
place of the record's original geometry for the relevant display.

The module implementation exposes methods including:

- `print_box()`;
- `default_grid_geom()`; and
- `get_grid_layer()`.

#### Parameters

```json
{
  "layer_options": [
    "kef_5 (layer_data_grid)",
    "original (layer_data_points)"
  ]
}
```

Each `layer_options` entry associates a geometry column with a MapServer
layer:

- the text before the parentheses is a column in `YOURTABLE_qgrids`; and
- the text inside the parentheses is the corresponding MapServer layer name.

In the example:

- `kef_5` is a geometry column in `YOURTABLE_qgrids`;
- `layer_data_grid` is the MapServer polygon layer used to display it;
- `original` stores the source geometry; and
- `layer_data_points` displays the original points.

A grid geometry requires a compatible MapServer layer. For example,
`layer_data_grid` must be a polygon layer if it displays polygon grids.

#### Grid table

The module creates `YOURTABLE_qgrids` if it does not already exist. The table
can then be extended with the geometry columns required by the project.

The module may also create an `update_grid_geoms` trigger and initial column
comments. These generated objects normally require project-specific review
and modification.

Set the user-facing names of grid options as column comments:

```sql
COMMENT ON COLUMN public.YOURTABLE_qgrids.original IS 'Original';
COMMENT ON COLUMN public.YOURTABLE_qgrids.kef_5 IS 'KEF 5×5';
```

Keep SQL identifiers consistent. For example, do not configure `kef_5` in the
module and create a column named `kef5`.

#### Trigger on the grid table

The following example invokes a project-specific grid-update function:

```sql
CREATE TRIGGER update_grid_geoms
BEFORE INSERT OR UPDATE ON public.YOURTABLE_qgrids
FOR EACH ROW
EXECUTE PROCEDURE public.update_qgrid_geoms_arg(
  '0.1',
  '0.1',
  't',
  't',
  't',
  't',
  '0.05'
);
```

> **Important:** The number and order of trigger arguments must exactly match
> the installed definition of `update_qgrid_geoms_arg()`. The historical
> example function below reads argument indexes up to `TG_ARGV[8]`, while the
> example trigger above supplies only seven arguments. Do not deploy these
> examples unchanged. Inspect the installed `grid_view.sql` and the database
> function, then provide every required argument.

#### Trigger on the source table

The source table requires a trigger to copy changes into the grid table:

```sql
CREATE TRIGGER qgrids
BEFORE INSERT OR DELETE OR UPDATE ON public.YOURTABLE
FOR EACH ROW
EXECUTE PROCEDURE insert_originalgeom_into_qgrids();
```

An example function body is:

```sql
BEGIN
  IF TG_OP = 'INSERT' THEN
    EXECUTE format(
      'INSERT INTO %I_qgrids (row_id, original) SELECT %L, %L::geometry',
      TG_TABLE_NAME,
      NEW.obm_id,
      NEW.obm_geometry
    );
    RETURN NEW;
  END IF;

  IF TG_OP = 'UPDATE' THEN
    EXECUTE format(
      'UPDATE %I_qgrids SET original = %L::geometry WHERE row_id = %L',
      TG_TABLE_NAME,
      NEW.obm_geometry,
      NEW.obm_id
    );
    RETURN NEW;
  END IF;

  IF TG_OP = 'DELETE' THEN
    EXECUTE format(
      'DELETE FROM %I_qgrids WHERE row_id = %L',
      TG_TABLE_NAME,
      OLD.obm_id
    );
    RETURN OLD;
  END IF;

  RETURN NULL;
END;
```

This is only the body of a trigger function, not a complete
`CREATE FUNCTION` statement.

#### Grid-update function

The following historical example demonstrates the intended operations:

```sql
DECLARE
  snap_x numeric := TG_ARGV[0];
  snap_y numeric := TG_ARGV[1];
  kef5 boolean := TG_ARGV[2];
  utm10 boolean := TG_ARGV[5];
  snap boolean := TG_ARGV[6];
  snap_polygon boolean := TG_ARGV[7];
  snap_polygon_size numeric := TG_ARGV[8];
BEGIN
  IF TG_OP = 'UPDATE' THEN
    IF kef5 THEN
      EXECUTE format(
        'SELECT geometry FROM shared."kef_5x5" WHERE ST_Within(%L::geometry, geometry)',
        NEW.original
      )
      INTO NEW.kef_5;
    END IF;

    IF snap THEN
      EXECUTE format(
        'SELECT ST_SnapToGrid(%L::geometry, %L, %L)',
        NEW.original,
        snap_x,
        snap_y
      )
      INTO NEW.snap;
    END IF;

    IF snap_polygon THEN
      EXECUTE format(
        'SELECT ST_Expand(ST_SnapToGrid(%L::geometry, %L, %L), %L)',
        NEW.original,
        snap_x,
        snap_y,
        snap_polygon_size
      )
      INTO NEW.snap_polygon;
    END IF;

    RETURN NEW;
  END IF;

  IF TG_OP = 'INSERT' THEN
    IF kef5 THEN
      EXECUTE format(
        'SELECT geometry FROM shared."kef_5x5" WHERE ST_Within(%L::geometry, geometry)',
        NEW.original
      )
      INTO NEW.kef_5;
    END IF;

    IF snap THEN
      EXECUTE format(
        'SELECT ST_SnapToGrid(%L::geometry, %L, %L)',
        NEW.original,
        snap_x,
        snap_y
      )
      INTO NEW.snap;
    END IF;

    IF snap_polygon THEN
      EXECUTE format(
        'SELECT ST_Expand(ST_SnapToGrid(%L::geometry, %L, %L), %L)',
        NEW.original,
        snap_x,
        snap_y,
        snap_polygon_size
      )
      INTO NEW.snap_polygon;
    END IF;

    RETURN NEW;
  END IF;

  RETURN NEW;
END;
```

This is also only a function body. The `utm10` variable is declared but is
not used by the shown implementation. Review and complete the function for
the project's required grid types.

#### Initial population

After the grid table and triggers are ready, existing source geometries can be
copied into an empty grid table:

```sql
INSERT INTO YOURTABLE_qgrids (row_id, original)
SELECT obm_id, obm_geometry
FROM YOURTABLE;
```

Example update for a snapped geometry:

```sql
UPDATE YOURTABLE_qgrids AS q
SET snap = source.snapped_geometry
FROM (
  SELECT
    obm_id,
    ST_SnapToGrid(obm_geometry, 0.13, 0.09) AS snapped_geometry
  FROM YOURTABLE
) AS source
WHERE q.row_id = source.obm_id;
```

Example update using polygons from a shared grid table:

```sql
UPDATE YOURTABLE_qgrids AS q
SET kef_5 = source.grid_geometry
FROM (
  SELECT
    data.obm_id,
    grid.obm_geometry AS grid_geometry
  FROM YOURTABLE AS data
  LEFT JOIN shared.kef_5x5 AS grid
    ON ST_Within(data.obm_geometry, grid.obm_geometry)
) AS source
WHERE q.row_id = source.obm_id;
```

In this example, `shared.kef_5x5` contains the predefined grid polygons.
Another geometry, such as `snap`, can be generated dynamically.

Run schema changes and bulk updates in a test environment first. Create a
database backup, verify spatial indexes, and check the behaviour for null,
invalid, boundary, and non-point geometries.

### `job_manager` validation jobs

The validation job manager configures background processes for a project.

On its administration page, administrators can configure:

- a simplified schedule containing minute, hour, and day values; and
- job-specific parameters in JSON format.

Adding a job registers it in the project's jobs table and can create template
files in the validation-module and jobs directories.

The availability and exact name of this component may depend on the installed
validation module. Background jobs run only if the project's `jobs.php`
runner is scheduled on the server.

**Parameters:** A list of background-job names.

#### `observation_lists`

The `observation_lists` job processes observation lists uploaded by the
mobile application.

Uploaded observations initially arrive in a temporary table. The job:

- populates `obm_observation_list_id`;
- calculates or copies the list start, end, and duration values; and
- copies complete lists into their target table.

Incomplete lists are skipped for later processing.

Job parameters:

- `list_start_column`: column storing the list start;
- `list_end_column`: column storing the list end;
- `list_duration_column`: column storing the duration;
- `only_time`: whether to store only the time instead of the full timestamp;
- `time_as_int`: whether to convert the time or duration to minutes.

Example:

```json
{
  "YOURTABLE": {
    "list_start_column": "time_of_start",
    "list_end_column": "time_of_end",
    "list_duration_column": "duration",
    "only_time": true,
    "time_as_int": true
  }
}
```

#### `incomplete_observation_lists`

The `incomplete_observation_lists` job handles lists that remain incomplete.

If the difference between the expected and received observations is within
the configured tolerance, the list can be processed by the next
`observation_lists` run and a system message is sent.

If the difference exceeds the tolerance, the job sends a system message but
leaves the list for manual processing.

Job parameters:

- `mail_to`: numeric role ID whose members receive the message;
- `diff_tolerance`: permitted difference before manual processing is required;
- `days_offset`: number of days to wait before processing the incomplete list.

Example:

```json
{
  "YOURTABLE": {
    "mail_to": 1265,
    "diff_tolerance": 2,
    "days_offset": 2
  }
}
```

Test notification recipients and job scheduling before relying on this
workflow in production.

### `join_tables`

The `join_tables` module displays related records on a data-sheet page.

The current documented implementation supports simple `LEFT JOIN` operations
with one equality condition per joined table.

**Parameters:**

```json
[
  {
    "table": "events",
    "join_on": [
      {
        "ref_field": "obm_id",
        "join_field": "patient_id"
      }
    ]
  },
  {
    "table": "measurements",
    "join_on": [
      {
        "ref_field": "obm_id",
        "join_field": "record_id"
      }
    ]
  }
]
```

For each joined table:

- `table` is the table to join;
- `ref_field` is the field in the current record; and
- `join_field` is the matching field in the joined table.

Use this module together with `additional_columns` where required. Ensure that
join fields are indexed and that users are authorised to access data from
every joined table.

### `list_manager`

The `list_manager` module manages reusable term lists for data uploads and
queries.

It provides:

- creation and editing of lists;
- association of lists with database tables and columns;
- generation of list contents from existing data;
- storage of list data in the database; and
- user feedback when a list operation fails.

The module uses a modal dialog for editing list values. Access to its
administrative functions should be limited to users who are permitted to
change upload and query vocabularies.

**Parameters:** None. The module uses its own user interface and
module-specific database objects.

### `massive_edit`

The `massive_edit` module allows authorised users to edit multiple selected
records from the map page through the file-upload interface.

Bulk changes can affect many records. Verify permissions, create a backup, and
test the edited file on a small selection before applying a large update.

**Parameters:** None.

### `move_project`

The `move_project` module moves a project to another OpenBioMaps server.

This is an experimental module. Before using it, create and verify backups and
check the compatibility of application versions, database extensions,
project files, users, modules, MapServer configuration, and secrets on the
destination server.

**Parameters:** None documented.

### `read_table`

The `read_table` module exposes a SQL table or view as a scrollable HTML table
through a unique link.

**Parameters:**

```json
[
  {
    "table": "schema.table_name",
    "label": "Displayed table name",
    "orderby": "column_name"
  }
]
```

Each entry contains:

- `table`: schema-qualified table or view name;
- `label`: user-facing label; and
- `orderby`: column used for the default ordering.

A unique or difficult-to-guess link is not sufficient access control. Verify
that the module enforces the intended project, group, and record-level
permissions.

### `results_asList`

The `results_asList` module displays query results as collapsible,
slide-like entries.

**Parameters:** None.

### `results_asGPX`

The `results_asGPX` module exports query results as a GPX file.

**Parameters:**

```json
{
  "name": "name_column",
  "description": [
    "description_column_1",
    "description_column_2"
  ]
}
```

The `name` column supplies the GPX feature name. Values from the
`description` columns are included in the feature description.

Only geometries compatible with the installed GPX exporter can be exported.

### `results_asCSV`

The `results_asCSV` module exports query results as a CSV file.

**Parameters:**

```json
{
  "sep": ",",
  "quote": "\""
}
```

- `sep` defines the field separator.
- `quote` defines the field-enclosure character.

Choose settings compatible with the software used to open the export. The
export must still enforce all applicable row- and column-level access rules.

### `results_asJSON`

The `results_asJSON` module exports query results as JSON.

**Parameters:** None.

### `results_asTable`

The `results_asTable` module displays query results as a full-screen HTML
table containing all available fields.

It provides:

- full record display;
- sortable columns; and
- links for viewing or editing records when the user has the required
  permissions.

Displaying every available field can be expensive for large result sets and
may expose fields that should be restricted. Configure access-control modules
and test the output for each user group.

**Parameters:** None.

### `results_asKML`

The `results_asKML` module exports query results as a KML file.

**Parameters:**

```json
{
  "name": "name_column",
  "description": [
    "description_column_1",
    "description_column_2"
  ]
}
```

The `name` column supplies the KML feature name. Values from the
`description` columns are included in the feature description.

### `results_buttons`

The `results_buttons` module adds controls for downloading, saving, sharing,
and bookmarking query results on the map page.

Available download formats depend on the corresponding enabled export
modules. These may include CSV, GPX, KML, SHP, and JSON.

The module can provide:

- download buttons;
- saved queries and results;
- saved spatial selections;
- bookmarks;
- sharing with users or other OpenBioMaps projects; and
- integration with `download_restricted`.

Example configuration:

```json
{
  "bookmarks": "off",
  "sharing": "off",
  "server_share": "on"
}
```

Configuration keys:

- `bookmarks` enables or disables query bookmarks;
- `sharing` enables or disables general sharing; and
- `server_share` enables or disables sharing with another server or project.

The documented defaults enable bookmarks and disable sharing and server
sharing. Download buttons appear when their corresponding export modules are
enabled.

When `download_restricted` is active, download availability also depends on
the request and approval workflow.

### `results_asStable`

The `results_asStable` module displays a compact, sortable result table on the
map page.

Unlike a full result table, it displays only the configured columns. It can
also include links to view or edit records when the user has the required
permissions.

**Parameters:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

The module name uses the historical spelling `results_asStable`; do not rename
it in the configuration.

### `results_specieslist`

The `results_specieslist` module summarises the species present in the current
query result.

It can display:

- species names;
- the number of records for each species;
- the number of recorded individuals; and
- alphabetical or taxonomic sorting options.

The columns used for species names and individual counts depend on the
project's schema and module implementation.

**Parameters:** None documented.

### `results_summary`

The `results_summary` module displays the total number of distinct records
returned by the current query.

It integrates with access rules so that restricted records are counted only
when the user is permitted to access them.

A summary count can itself disclose sensitive information. Test the module
with restricted records and each relevant access level.

**Parameters:** None.

### `results_table`

The `results_table` extension creates a full HTML table from query results.

Depending on the OpenBioMaps release, this functionality may be provided by a
module with another implementation name, such as `results_asTable` or
`results_asHtmlTable`. Use the exact module name shown by the installed module
administration page.

**Parameters:** None documented.

### `restricted_data`

The `restricted_data` module applies rule-based restrictions to project data.

The project must have correctly configured access rules and related database
objects. Test restrictions through the map page, data-sheet page, exports,
APIs, and direct module links.

**Parameters:** None.

### `spa_integration`

The `spa_integration` module integrates a single-page application with an
OpenBioMaps project.

It requires module-specific administration settings. Routing,
authentication, authorisation, static-resource paths, and direct browser
navigation must be tested after configuration.

**Parameters:** Managed through the module administration interface.

### `text_filter`

The `text_filter` module adds text filters to the map page and the query API.
It constructs the filtering portion of the SQL query from configured columns
and filter operators.

Example:

```json
[
  "common_name",
  "obm_taxon",
  "notes::colour_rings",
  "obm_datum",
  "obm_uploading_date",
  "obm_uploader_user",
  "data.abundance:nested(data.count):autocomplete",
  "data.count:values():",
  "obm_files_id",
  "species::autocomplete"
]
```

Entries can contain a column reference followed by module-specific modifiers,
such as:

- `autocomplete`;
- `values()`;
- `nested(...)`; or
- a label or secondary field separated with `::`.

The syntax is compact and version-dependent. Copy known working expressions
carefully, use only trusted database identifiers, and test every filter
through both the map page and query API.

### `text_filter2`

The `text_filter2` module provides advanced taxonomic and general text
filters. Like `text_filter`, it contributes conditions to the SQL query.

**Parameters:**

```json
{}
```

Further settings may be managed through the module's user interface or
project-specific configuration.

### `transform_data`

The `transform_data` module transforms record values before they are displayed
in result areas or included in supported exports.

Available transformations include:

- `geom`: creates a clickable geometry link that opens the location on the
  map;
- `geom_nolink`: displays simplified WKT without a link;
- `geom_wkt`: displays the normal WKT representation;
- `date_yearonly`: extracts the year from a date;
- `translate`: translates predefined text constants into user-facing text;
- `obslistlink`: creates a link from an observation-list identifier; and
- `uplid`: applies the upload-identifier transformation supported by the
  module.

Example:

```json
{
  "obm_geometry": "geom",
  "other_geometry": "geom_nolink",
  "obm_uploading_id": "uplid",
  "date_time_field": "date_yearonly",
  "method": "translate",
  "obm_observation_list_id": "obslistlink"
}
```

Each key is a database column and each value is the transformation applied to
that column.

Transformations affect presentation, not the underlying stored value. Verify
that transformed links and values do not reveal restricted data.

### `pds_test_evaluator`

The `pds_test_evaluator` module records and evaluates manual PDS form-upload
test runs. On initialisation it creates its tables in the project schema. The
PDS mobile form test catalogue must be installed with `protocol_installer`
first; otherwise enabling the module fails with an error message. The
administration page prepares test scenarios and reference choices.

**Parameters:** None.

### `custom_data_check`

The `custom_data_check` module runs project-specific checks on uploaded data.
The parameter value is a semicolon-separated list of check function names.
Check functions are implemented in the module source, in the block reserved for
custom functions, and return `true` or `false`. The `colour_rings` check is
built in as an example. Checks whose name starts with `select_list(` are always
accepted.

The module is a framework with minimal logic and no ready-to-use checks:
adding a check requires editing the module source.

**Parameters:**

```text
check_function_1;check_function_2
```

### `results_asHtmlTable`

The `results_asHtmlTable` module renders the query result as a simple HTML
table. It reads at most 2000 rows from the temporary query table of the
session. The module is not used by the standard result views; use
`results_asTable` instead.

**Parameters:** None.

### `results_asPDF`

The `results_asPDF` module exports query results as a PDF report generated with
mPDF. The report is selected by name and rendered from the Mustache template
`views/documents/report_<name>.mustache` of the application theme. The default
theme provides the `observations` report.

Additional reports, or custom data preparation for a report, can be added in
the project file `local/includes/modules/results_asPDF_CM.php`, which defines
the `results_asPDF_CM` class. A static `prepare_report_<name>($de)` method
returns the template data, and an optional static `config_<name>()` method
returns the mPDF configuration.

**Parameters:** None.

### `results_asSHP`

The `results_asSHP` module exports query results as ESRI Shapefiles in a ZIP
archive. A geometry column must be defined for the queried table. One
Shapefile is written for each geometry type found in the result, and the files
are converted with `ogr2ogr` (GDAL) in the EPSG:4326 reference system.

The `ogr2ogr` command must be installed on the server. If it is not on the
`PATH`, set the `GDAL_PATH` constant in the system configuration to the
directory that contains it.

**Parameters:** None documented.

### `service_envimap`

The `service_envimap` module connects the project to the EnviMap service. It
sends the GeoJSON of the area of interest, lists the available services,
fetches the status of a service request, and shows service access links and
buttons in the interface. The administration page is available to project
masters only.

The API base URL can be set with the `ENVIMAP_API_BASE` constant. If it is not
defined, `https://api.envimap.hu/v1/` is used.

**Parameters:** None documented.

### `snap_to_grid`

The `snap_to_grid` module is deprecated and fully replaced by `grid_view`. For
visitors who are not logged in, it rounds the geometry column with
`ST_SnapToGrid`. The parameter is two semicolon-separated grid sizes; the
defaults are `0.13;0.09`. Do not use it in new projects.

**Parameters:**

```text
x_size;y_size
```

### `ebp`

The `ebp` module is a EuroBirdPortal module skeleton. It contains the template
structure of a table-level module (translatable strings, AJAX dependencies,
external dependencies and an `init` method with example SQL) but no
production functionality. Do not enable it in a production project.

**Parameters:** None.

## Background helper scripts

The following files in the module directory are not modules. They are
command-line scripts started by the application in the background, and they
must not be enabled or called manually.

### `fill_table_with_rows`

Asynchronous worker of `results_asTable`. It fills the result table with rows
in batches after the first rows were returned to the browser. The first batch
is 80 rows by default. The script runs only from the command line, and it
receives the session variables through an environment variable that is removed
on start.

### `fill_stable_with_column`

Asynchronous worker of `results_asStable`. It fills the columns of the stable
result table in the background and restores the session from a base64-encoded
environment variable.

### `computation_fork`

Command-line script of the `computation` module that packages a computational
package from the `computational_packages` directory of the project, uploads it
to the configured working server and reports the progress through the
application cache. The package settings are read from the
`computation_conf.yml` file of the package, and the working server URL is saved
to `computation_state.yml`. The PHP YAML extension is required.

## Deployment checklist

Before enabling a module in production:

1. Confirm that the module is included in the installed OpenBioMaps version.
2. Validate its JSON parameters.
3. Identify any required modules, database objects, MapServer layers, jobs,
   external services, or PHP extensions.
4. Review module access and group access.
5. Test with public, authenticated, group-member, and administrator accounts.
6. Test direct URLs, exports, and APIs for access-control bypasses.
7. Check application, PHP, PostgreSQL, and background-job logs.
8. Back up the project before enabling modules that alter database objects or
   perform bulk updates.
9. Document the configuration and the procedure for disabling or rolling back
   the module.
10. Repeat the tests after an OpenBioMaps upgrade.
