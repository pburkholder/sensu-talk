
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



````
sensu_check 'check_carbon_relay_process' do
  command '/usr/lib/nagios/plugins/check_procs -c 8:8 -C carbon-relay.py'
  handlers %w(hipchat daytime_pagerduty)
  subscribers ['carbon-relay']
  interval 60
  additional(
    notification: 'carbon-relay processes should number exactly 8',
    occurrences: 3
  )
end

# the above check may pass if upstart is thrashing, so also check
# if we can connect to carbon-relay-a and carbon-relay-h
sensu_check 'check_carbon_relay_tcp' do
  command '/usr/lib/nagios/plugins/check_tcp -H localhost -p 3004 -w 0.50 -c 1.00 && ' +
   '/usr/lib/nagios/plugins/check_tcp -H localhost -p -p 3021 -w 0.50 -c 1.00'
  handlers %w(hipchat daytime_pagerduty)
  subscribers ['carbon-relay']
  interval 60
  additional(
    notification: 'carbon-relay should listen on 3004 and 3021',
    occurrences: 3
  )
end
```

Add to worker.rb:


