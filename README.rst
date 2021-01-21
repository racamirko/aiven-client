Aiven Client |BuildStatus|_
###########################

.. |BuildStatus| image:: https://travis-ci.org/aiven/aiven-client.png?branch=master
.. _BuildStatus: https://travis-ci.org/aiven/aiven-client

Aiven is a next-generation managed cloud services platform.  Its focus is in
ease of adoption, high fault resilience, customer's peace of mind and
advanced features at competitive price points.  See https://aiven.io/ for
more information about the backend service.

aiven-client (``avn``) is the official command-line client for Aiven.

.. contents::


.. _platform-requirements:

Getting Started
===============

Requirements:

*  Python 3.6 or later

*  Requests_

*  For Windows and OSX, certifi_ is also needed

.. _`Requests`: http://www.python-requests.org/
.. _`certifi`: https://certifi.io/

.. _installation:

Install from PyPi
-----------------

Pypi installation is the recommended route for most users::

  $ python3 -m pip install aiven-client


Build an rpm Package
--------------------

It is also possible to build an rpm::

  $ make rpm

Check Installation
------------------

To check that the tool is installed and working, run it without arguments::

  $ avn

If you see usage output, you're all set.

  **Note:** On Windows you may need to use ``python3 -m aiven.client`` instead of ``avn``.

Log In
------

The simplest way to use Aiven CLI is to authenticate with the username and
password you use on Aiven::

  $ avn user login <you@example.com>

The command will prompt you for your password.

.. _help-command:
.. _basic-usage:

Usage
=====

Some handy hints that work with all commands:

*  The ``avn help`` command shows all commands and can _search_ for a command,
   so for example ``avn help kafka topic`` shows commands with kafka *and*
   topic in their description.

*  Passing ``-h`` or ``--help`` gives help output for any command. Examples:
   ``avn --help`` or ``avn service --help``.

*  All commands will output the raw REST API JSON response with ``--json``,
   we use this extensively ourselves in conjunction with
   `jq <https://stedolan.github.io/jq/>`__.


.. _login-and-users:

Authenticate: Logins and Tokens
-------------------------------

Login::

  $ avn user login <your@email>

Logout (revokes current access token, other sessions remain valid)::

  $ avn user logout

Expire all authentication tokens for your user, logs out all web console sessions, etc.
You will need to login again after this.::

 $ avn user tokens-expire

Manage individual access tokens::

 $ avn user access-token list
 $ avn user access-token create --description <usage_description> [--max-age-seconds <secs>] [--extend-when-used]
 $ avn user access-token update <token|token_prefix> --description <new_description>
 $ avn user access-token revoke <token|token_prefix>

Note that the system has hard limits for the number of tokens you can create. If you're
permanently done using a token you should always use ``user access-token revoke`` operation
to revoke the token so that it does not count towards the quota.

Alternatively, you can add 2 JSON files, first create a default config in ``~/.config/aiven/aiven-credentials.json`` containing the JSON with an ``auth_token``::

  {
      "auth_token": "ABC1+123...TOKEN==",
      "user_email": "your.email@aiven.com"
  }

Second create a default config in ``~/.config/aiven/aiven-client.json`` containing the json with the ``default_project``::

  {"default_project": "yourproject-abcd"}

.. _clouds:

Choose your Cloud
-----------------

List available cloud regions::

  $ avn cloud list

.. _projects:

Working with Projects
---------------------

List projects you are a member of::

  $ avn project list

Project commands operate on the currently active project or the project
specified with the ``--project NAME`` switch. The active project cab be changed
with the ``project switch`` command::

  $ avn project switch <projectname>

Show active project's details::

  $ avn project details

Create a project and set the default cloud region for it::

  $ avn project create myproject --cloud aws-us-east-1

Delete an empty project::

  $ avn project delete myproject

List authorized users in a project::

  $ avn project user-list

Invite an existing Aiven user to a project::

  $ avn project user-invite somebody@aiven.io

Remove a user from the project::

  $ avn project user-remove somebody@aiven.io

View project management event log::

  $ avn events

.. _services:

Explore Existing Services
-------------------------

List services (of the active project)::

  $ avn service list

List services in a specific project::

  $ avn service list --project proj2

List only a specific service::

  $ avn service list db1

Verbose list (includes connection information, etc.)::

  $ avn service list db1 -v

Full service information in JSON, as it is returned by the Aiven REST API::

  $ avn service list db1 --json

Only a specific field in the output, custom formatting::

  $ avn service list db1 --format "The service is at {service_uri}"

View service log entries (most recent entries and keep on following logs, other options can be used to get history)::

  $ avn service logs db1 -f

.. _launching-services:

Launch Services
---------------

View available service plans::

  $ avn service plans

Launch a PostgreSQL service::

  $ avn service create mydb -t pg --plan hobbyist

View service type specific options, including examples on how to set them::

  $ avn service types -v

Launch a PostgreSQL service of a specific version (see above command)::

  $ avn service create mydb96 -t pg --plan hobbyist -c pg_version=9.6

Update a service's list of allowed client IP addresses. Note that a list of multiple
values is provided as a comma separated list::

  $ avn service update mydb96 -c ip_filter=10.0.1.0/24,10.0.2.0/24,1.2.3.4/32

Open psql client and connect to the PostgreSQL service (also available for InfluxDB)::

  $ avn service cli mydb96

Update a service to a different plan AND move it to another cloud region::

  $ avn service update mydb --plan startup-4 --cloud aws-us-east-1

Power off a service::

  $ avn service update mydb --power-off

Power on a service::

  $ avn service update mydb --power-on

Terminate a service (all data will be gone!)::

  $ avn service terminate mydb

Managing service users
----------------------

Some service types support multiple users (e.g. PostgreSQL database users).

List, add and delete service users::

  $ avn service user-list
  $ avn service user-create
  $ avn service user-delete

For Redis services running version 6 or above, it's possible to create users with ACLs_::

  $ avn service user-create --username new_user --redis-acl-keys "prefix* another_key" --redis-acl-commands "+set" --redis-acl-categories "-@all +@admin" my-redis-service

.. _`ACLs`: https://redis.io/topics/acl

Service users are created with strong random passwords.

.. _teams:

Working with Teams
------------------

List account teams::

  $ avn account team list <account_id>

Create a team::

  $ avn account team create --team-name <team_name> <account_id>

Delete a team::

  $ avn account team delete --team-id <team_id> <account_id>

Attach team to a project::

  $ avn account team project-attach --team-id <team_id> --project <project_name> <account_id> --team-type <admin|developer|operator|read_only>


Detach team from project::

  $ avn account team project-detach --team-id <team_id> --project <project_name> <account_id>

List projects associated to the team::

  $ avn account team project-list --team-id <team_id> <account_id>

List members of the team::

  $ avn account team user-list --team-id <team_id> <account_id>

Invite a new member to the team::

  $ avn account team user-invite --team-id <team_id> <account_id> <user@email>

See the list of pending invitations::

  $ avn account team user-list-pending --team-id <team_id> <account_id>

Remove user from the team::

  $ avn account team user-delete --team-id <team_id> --user-id <user_id> <account_id>

Extra Features
==============

.. _shell-completions:

Autocomplete
------------

avn supports shell completions. It requires an optional dependency: argcomplete. Install it::

  $ python3 -m pip install argcomplete

To use completions in bash, add following line to `~/.bashrc`::

  eval "$(register-python-argcomplete avn)"

For more information (including completions usage in other shells) see https://kislyuk.github.io/argcomplete/.

Keep Reading
============

We maintain some other resources that you may also find useful:

* `Command Line Magic with avn <https://aiven.io/blog/command-line-magic-with-the-aiven-cli>`__
* `Managing Billing Groups via CLI <https://help.aiven.io/en/articles/4720981-using-billing-groups-via-cli>`__
