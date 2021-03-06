# trac-webhook-plugin

Plugin to ```POST``` Trac changes to a webhook endpoint.

The goal is to provide a complete dataset for as many events as possible.  I send the events to Node-RED for handling which makes it easy to send notifications to slack, make rpc calls, re-publish the event elsewhere (another webhook, messages brokers such as mqtt, amqp, redis or others), etc, etc.

Currently 2 resource types are supported:
 1. Tickets
 1. WikiPage

## Installation

Requirements:

* [Requests](https://pypi.python.org/pypi/requests)

```
python setup.py bdist_egg;
cp dist/*.egg /path/to/plugins
```

Install and enable the plugin in `trac.ini`:

```ini
[components]
webhook_notification.* = enabled
```

Configuration in trac.ini:

```ini
[webhook]
secret = randomstring
url = https://host/webhook/path
username = HTTP Auth Username
password = HTTP Auth Password
ssl_verify = true|false # default is true, if false self-signed certs can be used
intelligent_ticket_change_action = true|false # default is false, if true newly set status on ticket will be marked as the event action
```

Some notes on the configuration:

* The `secret` must be a random string which is configured also in the
  receiving endpoint. It is used to generate a HMAC-SHA1 hex digest of the
  body using `secret` as the key. The digest is sent in the
  `X-WebHook-Signature` HTTP header.

## Data Format
All events publish the following standard fields: 
```
{
   "action":"see list of per-realm actions",
   "realm":"wiki|ticket",
   "invoke_url":"url that generated the event",
   "project":{
      "admin":"admin@awesome.com",
      "description":"Awesome Project",
      "name":"Awesome",
      "base_url":"https://trac.awesome.com",
      "url":"https://www.awesome.com",
      "icon":"site/favicon.ico"
   },
   "resource_url":"url to either the target ticket or wiki page",
   "user":{
      "email":"user@awesome.com",
      "name":"user who instigated the change",
      "username":"user"
   },
   "data":{...}
}
```
If the realm is `ticket` then `data.ticket` will include all the ticket details.

If the realm is `wiki` then `data.page` will include all the wiki page details.

If the action is an `attachment` action then the result will include the appropriate key for either `ticket` or `wiki` **and** additionally a `data.attachment` key with the details of the `attachment`.

Various *action specific* `data` keys (eg: if a `ticket` `changed` event is triggered there will be a `data.old_values` key with all the changes).


## Development

```python setup.py develop --multi-version --exclude-scripts --install-dir /path/to/plugins/```

## Acknowledgements

This plugin is based on the [trac-webhook-plugin](https://github.com/aperezdc/trac-webhook-plugin),
which is based on the [Slack Notification plugin](https://github.com/mandic-cloud/trac-slack-plugin),
which is based on the [Irker Notification plugin](https://github.com/Southen/trac-irker-plugin).
Lots of thanks go to their authors!

## TODO

Implement:
 1. [IMilestoneChangeListener](https://trac.edgewall.org/wiki/TracDev/PluginDevelopment/ExtensionPoints/trac.ticket.api.IMilestoneChangeListener)
 1. [IRepositoryChangeListener](https://trac.edgewall.org/wiki/TracDev/PluginDevelopment/ExtensionPoints/trac.versioncontrol.api.IRepositoryChangeListener)

## License

```
Copyright (c) 2017, Travis Glenn Hansen <travisghansen@yahoo.com>
Copyright (c) 2016, Adrián Pérez de Castro <aperez@igalia.com>
Copyright (c) 2014, Sebastian Southen
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in
   the documentation and/or other materials provided with the
   distribution.
3. The name of the author may not be used to endorse or promote
   products derived from this software without specific prior
   written permission.

THIS SOFTWARE IS PROVIDED BY THE AUTHOR `AS IS'' AND ANY EXPRESS
OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
