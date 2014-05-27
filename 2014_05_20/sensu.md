## Sensu & Self-service Monitoring

* Overview of architecture and capabilities
* What comprises a check
- How results are handled
- Introducing `sensu-runner`

---
## Sensu Architecture

--
### Anatomy of a Check

--
### Handling Results

---
## Introducting sensu-runner

Idea: Supplement the server-driven, Chef-configured checks
with self-service check suites:

    sensurun carbon-relay
    OK spawn_eight_carbons: PROCS OK: 8 processes with command name 'carbon-relay.py'
    OK carbon_listen_3004: TCP OK - 0.001 second response time on port 3004
    OK connect_carboncache_2014: TCP OK - 0.001 second response time on port 2014

--
## Input format

`/etc/sensu/runners/carbon-relay`:
    checks:
      spawn_eight_carbons: /usr/lib/nagios/plugins/check_procs -c 8:8 -C carbon-relay.py
      carbon_listen_3004: /usr/lib/nagios/plugins/check_tcp -H localhost -p 3004
      connect_carboncache_2014: /usr/lib/nagios/plugins/check_tcp -H carbon0.ops.audax.in -p 2014
    handlers:
      hipchat_yoloswag

--
## Subscriptions

* chef-role
** including zs-role::(role) e.g., `lift`
** true chef roles, e.g., `swagger`
* chef-cookbook tags
** zs-api/recipes/client.rb
** `tag 'sensu:haproxy_client'`

---
## Use Cases

* Interactive Use:
      sensurun carbon-relay
      OK spawn_eight_carbons: PROCS OK: 8 processes with command name 'carbon-relay.py'
* Development Sensu integration:
** sensu-client runs the check every 2 minutes
** results are published to the Sensu Aggregation framework
** sensu-server asks for check_aggregation every 2 minutes
** 2 failures results in handler triggering
* Prod Sensu integration

-- 
## What this means to you





Note: This was supposed to be where I introduced you all 
to a bunch of Chef cookbook coding. 