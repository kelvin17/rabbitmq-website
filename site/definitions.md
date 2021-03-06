<!--
Copyright (c) 2007-2020 VMware, Inc. or its affiliates.

All rights reserved. This program and the accompanying materials
are made available under the terms of the under the Apache License,
Version 2.0 (the "License”); you may not use this file except in compliance
with the License. You may obtain a copy of the License at

https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Schema Definition Export and Import

## <a id="overview" class="anchor" href="#overview">Overview</a>

Nodes and clusters store information that can be thought of schema, metadata or topology.
Users, vhosts, queues, exchanges, bindings, runtime parameters all fall into this category.

Definitions are stored in an internal database and replicated across all cluster nodes.
Every node in a cluster has its own replica of all definitions. When a part of definitions changes,
the update is performed on all nodes in a single transaction. This means that
in practice, definitions can be exported from any cluster node with the same result.

Definitions can be [exported](#export) to a file and then [imported](#imported) into another cluster or
used for schema backup.

A definition file contains definitions of all broker objects (queues,
exchanges, bindings, users, virtual hosts, permissions and
parameters).

Definition import on node boot is the recommended way of pre-configuring nodes at deployment time.

## <a id="export" class="anchor" href="#export">Definition Export</a>

Definitions are exported as a JSON file in a number of ways.
 
 * [`rabbitmqctl export_definitions`](/cli.html) is the only option that does not require [management plugin](/management.html) to be enabled
 * The `GET /api/definitions` API endpoint
 * [`rabbitmqadmin export`](/management-cli.html) which uses the above HTTP API endpoint
 * There's a definitions pane on the Overview page

Definitions can be exported for a specific [virtual host](/vhosts.html) or the entire cluster (all virtual host).
When definitions are exported for just one virtual host, some information (contents of the other
virtual hosts or users without any permissions to the target virtual host) will be
excluded from the exported file.

Exported user data contains password hashes as well as [password hashing function](/passwords.html) information. While brute forcing passwords with hashing functions such as SHA-256 or SHA-512 is not a completely trivial task,
user records should be **considered sensitive information**.

To export definitions using [`rabbitmqctl`](/cli.html), use `rabbitmqctl export_definitions`:

<pre class="lang-ini">
# Does not require management plugin to be enabled, new in RabbitMQ 3.8.2
rabbitmqctl export_definitions /path/to/definitions.file.json
</pre>

`rabbitmqadmin export` is very similar but uses the HTTP API and is compatible
with older versions:

<pre class="lang-ini">
# Requires management plugin to be enabled
rabbitmqadmin export /path/to/definitions.file.json
</pre>

## <a id="import" class="anchor" href="#import">Definition Import</a>

To import definitions using [`rabbitmqctl`](/cli.html), use `rabbitmqctl import_definitions`:

<pre class="lang-ini">
# Does not require management plugin to be enabled, new in RabbitMQ 3.8.2
rabbitmqctl import_definitions /path/to/definitions.file.json
</pre>

`rabbitmqadmin import` is its HTTP API equivalent:

<pre class="lang-ini">
# Requires management plugin to be enabled
rabbitmqadmin import /path/to/definitions.file.json
</pre>

It is also possible to use the HTTP API endpoint directly.
Here's an example that contacts a local node at `localhost:15672` using `curl`
and [default user credentials](/access-control.html):

<pre class="lang-bash">
curl -H "Accept:application/json" -u guest:guest "http://localhost:15672/api/definitions"
</pre>

## <a id="import-on-boot" class="anchor" href="#import-on-boot">Definition Import at Node Boot</a>

Most recent releases support definition import directly in the core,
without the need to [preconfigure](/plugins.html#enabled-plugins-file) the [management plugin](/management.html).

To import definitions from a local file on node boot,
set the `load_definitions`config key
to the path of a previously exported JSON file containing
the definitions that should be imported on node boot:

<pre class="lang-ini">
# Does not require management plugin to be enabled, new in RabbitMQ 3.8.2
load_definitions = /path/to/definitions/file.json
</pre>

Older versions only provided a similar feature as part
of the management plugin:

<pre class="lang-ini">
# Requires management plugin to be enabled at the time of node boot
management.load_definitions = /path/to/definitions/file.json
</pre>

The definitions in the file will not overwrite anything already in the broker.
However, if a blank (uninitialised) node imports a definition file, it will
not create the default virtual host and user.