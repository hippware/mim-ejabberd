### Module Description
This module implements [XEP-0313 (Message Archive Management)](http://xmpp.org/extensions/attic/xep-0313.html). It enables a service to store all users messages for one-to-one chats as well as group chats (MUC, MultiUser Chat). It uses [XEP-0059: Result Set Management](http://xmpp.org/extensions/xep-0059.html) for paging. It is a highly customizable module, that requires some skill and knowledge to operate properly and efficiently.

Configure MAM with different storage backends:

* ODBC (RDBMS, like MySQL, PostgreSQL, MS SQL Server)
* Riak KV (NOSQL)

### Configure MAM with ODBC backend

#### Options
* `mod_mam_odbc_prefs`, `mod_mam_mnesia_prefs`:
Consider the process as a kind of recipe. For each step you can enable none ("optional"), one ("single") or more ("multi") modules, according to instructions. Provided there are any, please use the options described in a specific step. All config parameters are boolean, so you can enable them by adding an atom to the configuration list, e.g. `{mod_mam_odbc_arch, [pm, no_writer]}`.

##### Step 1 (multi)
* `*mod_mam` + `mod_mam_odbc_arch`: Enables support for one-to-one messages archive.
* `mod_mam_muc` + `mod_mam_muc_odbc_arch`: Enables support for group chat messages archive.

If you haven't chosen any of the above, skip the next part.

**Options:**

* `mod_mam_muc`:
    * `host` (optional, default: `"conference.@HOST@"`): MUC host that will be archived; **Warning:** If you are using MUC Light, make sure this option is set to MUC Light domain.
* `mod_mam_odbc_arch`:
    * `pm` (mandatory when `mod_mam` enabled): Enable archiving one-to-one messages
    * `muc` (optional): Enable group chat archive, mutually exclusive with `mod_mam_muc_odbc_arch`. **Not recommended**, `mod_mam_muc_odbc_arch` is more efficient.
    * `simple`: Same as `{simple, true}`
    * `{simple, true}`: Store messages in XML and full JIDs. Archive MUST be empty to change this option.
    * `{simple, false} (default)`: Store messages and JIDs in internal format.
* `mod_mam_odbc_arch`, `mod_mam_muc_odbc_arch`:
    * `no_writer`: Disables default synchronous, slow writer and uses async one (step 5 & 6) instead.
    * `{simple, true}`: Store messages in XML and full JIDs. Archive MUST be empty to change this option.
    * `{simple, false} (default)`: Store messages and JIDs in internal format.

##### Step 2 (mandatory)
* `mod_mam_odbc_user`: Maps archive ID to integer.

**Options**

* `pm`: Mandatory when `mod_mam` enabled.
* `muc`: Mandatory when `mod_mam_muc` enabled.

##### Step 3 (optional, recommended)
* `mod_mam_cache_user`: Enables Archive ID to integer mappings cache.

**Options**

* `pm`: Optional, enables cache for one-to-one messaging, works only with `mod_mam` enabled.
* `muc`: Optional, enables cache for group chat messaging, works only with `mod_mam_muc` enabled.

##### Step 4 (single, optional)
Skipping this step will make `mod_mam` archive all the messages and users will not be able to set their archiving preferences. It will also increase performance.

* `mod_mam_odbc_prefs`: User archiving preferences saved in ODBC. Slow and not recommended, but might be used to simplify things and keep everything in ODBC.
* `mod_mam_mnesia_prefs`: User archiving preferences saved in Mnesia and accessed without transactions. Recommended in most deployments, could be overloaded with lots of users updating their preferences at once. There's a small risk of inconsistent (in a rather harmless way) state of preferences table. Provides best performance.

**Options:** (common for all three modules)

* `pm`: Optional, enables MAM preferences for user-to-user messaging, works only with `mod_mam` enabled.
* `muc`: Optional, enables MAM preferences for group chat messaging, works only with `mod_mam_muc` enabled.

##### Step 5 (single, optional, recommended, requires `mod_mam` module enabled and `no_writer` option set in `mod_mam_odbc_arch`)

Enabling asynchronous writers will make debugging more difficult.

* `mod_mam_odbc_async_pool_writer`: Asynchronous writer, will insert batches of messages, grouped by archive ID.

**Options:** (common for both modules)

* `pm`: Optional, enables the chosen writer for one-to-one messaging, works only with `mod_mam` enabled.
* `muc`: Optional, enables the chosen writer for group chat messaging, use only when `mod_mam_odbc_arch` has `muc` enabled. **Not recommended**.

##### Step 6 (single, optional, recommended, requires `mod_mam_muc` module enabled and `no_writer` option set in `mod_mam_muc_odbc_arch`)

Enabling asynchronous writers will make debugging more difficult.

* `mod_mam_muc_odbc_async_pool_writer`: Asychronous writer, will insert batches of messages, grouped by archive ID.

### Configure MAM with Riak KV backend

In order to use Riak KV as the backend for one-to-one archives, the following configuration must be used:

```erlang
{mod_mam, []}.
{mod_mam_riak_timed_arch_yz, [pm]}.
```

To archive both one-to-one and group chat (MUC, Multi-User Chat) messages use this configuration instead:

```erlang
{mod_mam, []}.
{mod_mam_muc, []}.
{mod_mam_riak_timed_arch_yz, [pm, muc]}.
```

The Riak KV backend for MAM stores messages in weekly buckets so it's easier to remove old buckets.
Archive querying is done using Riak KV 2.0 [search mechanism](http://docs.basho.com/riak/2.1.1/dev/using/search/)
called Yokozuna. Your instance of Riak KV must be configured with Yokozuna enabled.

This backend works with Riak KV 2.0 and above, but we recommend version 2.1.1.

### `mod_mam` options

- `add_archived_element`: add `<archived/>` element from MAM v0.2
- `is_complete_message`: module name implementing is_complete_message/3 callback.
  This callback returns true if message should be archived.

### Example configuration

Default configuration for `mod_mam`:

```erlang
{mod_mam, []}.
```

It's expanded to:

```erlang
{mod_mam, [{add_archived_element, false},
           {is_complete_message, mod_mam_utils}]}
```
