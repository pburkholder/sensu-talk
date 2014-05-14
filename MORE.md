
Example - testing Graphite carbon-relay

1) There should be 8 carbon-relay processes running
2) There should be tcp connections on port 3004,3005,...

Plugins: are the atomic unit of a check.

Image: plugs OR "This end up" image pointing down

Local install

`brew install nagios-plugins`

Plugins are available from sensu-community or nagios-plugins or roll-your-own

Called a plugin because it can plugin as long as the exit status code
convention is obeyed:
...

check_proc, check_tcp, check_http

`check_procs -c 8:8 -C carbon-relay.py`

