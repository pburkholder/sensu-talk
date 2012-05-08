sensu-talk
==========

How to approach the project
---------------------------

What do we want and how do we get there?

- Do the big things first
- Do the hard things first
- Don't let Sensu seem sucky
- Do the most useful things first


Reprioritize tasks and add time estimates (on paper, then Jira).  Use colors and numbers.  

The point of monitoring
-----------------------

mean-time-to-failure
mean-time-to-diagnosis
mean-time-to-resolution
availability

Screw that.

- What's happening
- What's happened

Or

Alert us when things are going wrong

Trend data to predict and diagnose

Key characteristics of monitoring


Audax only: Replace RightScale: RS

Talk
====


Good Evening

So, unless I'm mistaken more of our DevOpsDC talks have been dealt with
monitoring than any other area of our work.  Not as much time has been
devoted to, say, automated testing, configuration management, culture,
career development, operating systems, database or cloud providers to name a
a few.

Is this because "Monitoring Sucks"?

I would argue that it is more that monitoring is the technically most
"Idiosyncratic" thing that we have to concern ourselves with as the
developers of onlines systems, and the operations folks how run said
software.   (Say why it's more idiosyncratic: varies along multiple
dimensions: time, office-hours, after-hours, during deploys/maintenance; space
(which environment or role the monitored client occupies); and xxxxx the
distinctiveness of your application needs; and finally it's coupled back into
itself in terms of service dependencies and meta-monitoring.)


Wherever we sit in relation to this mess of running stuff, we'll
generally expect two or three things from the monitoring we have

1) What's happening?
(screen shot of an alert)

2) What's happened?
(screen shot of a trending system)

That is, we need to be informed when something is going south, preferably
before the impact it discernible to our customers so we can take remedial
actions. 

And we need ways of gathering and visualizing numeric metrics so we anticipate
future needs and look for correlations that lead us the cause of present (or
past problems)

System and application log aggregation is a third tool of what might be
considered a tripod, but they are generally distinct from the first two.

If you want to delve into monitoring considerations at a higher level I
recommend Limoncelli and Hogan; and Patrick Debois;  For our purposes I think
how Sensu fits into the monitoring universe can be well illustrated by a case
study consideration: namely, the monitoring needs we face at Audax Health in
running Careverge:

The Problem

At Audax Health we run health consumer social network called Careverge, and the
core architecture should be not unfamiliar to most of you.  Web traffic to 
www.careverge.com will 

a) hit an AWS ELB
b) an elastic array of web nodes
c) an internal LB
d) an elastic array of API nodes
e) a MongoDB server replication set
f) miscellaneous servers that provide Careverge services (recommendations) or
dev/infra services (puppet, jenkins, graphite, logging, monitoring)

And then smaller copies of that in various environments:  dev, test, and RC.
All of this is managed by Puppet, with some help from MCollective.

The way I've generally tackled monitoring over the last few gigs has been
with the Nagios monitoring framework because, despite the age of some of the
code, Ethan Galstad has gotten a lot of things right that I've seen utterly
absent from recent "enterprise-class" products (don't go into MySQL monitor,
Hyperic)

For the developers among us, the architecture of the Nagios framework looks
something like this

TKTK nagios architecture
TKTK nagios configuration language

While there's been a lot of discussion in DevOps circles about "monitoring
sucks" and to some extent I'd been wondering if that was people
misunderstanding how to use the object/templating features of Nagios, because
I've seen it woefully under-utilized in the past.  In particular, I wanted to
draw on Jordan Sissel's work describing monitoring at Loggly

(References: Patrick Debois monitoring roundup, Jordan Sissel at Loggly)

* A new node comes up and becomes a Puppet client
* Puppet stores configuration data in a MySQL DB using the 'store configs' extension to Puppet
* The client would get an nrpe.conf file based on the facts for that node.
* The next configuration run across the Nagios server would utilize the
  'exported resources' resources feature to update the hosts file.
** Puppet supports nagios\_host, nagios\_service, etc as first-class resources
** Show exported resources Puppet code
** Show example stanza for a new recommendation host
* Since the new host is a member of correct Nagios HostGroups, it would have
  the appropriate services monitored

Now you can probably guess at some of the pitfalls of this approach.

One caveat: default storeconfig can be a real resource pig in Puppet with
somewhere along the line for 10,000 inserts being done for each client
'thin\_storeconfigs' is much better and completely sufficient for storing
node data sufficient for exported node resources.

The obvious issue is the lag between node instantiation and the convergence of
the Nagios server.  Requiring a high-frequency of Puppet runs is resource
intensive, triggering runs from node instantiation is yet another layer of
complexity to build in, and in either case you risk frequent interruptions in
your monitoring while the Nagios server restarts to pick up the new
configuration, or worse, going down because somehow an error or inconsistency
appeared in the configuration.

For example, the hostgroups we assigned to each host was based on the role
parameter assigned to the node.  E.g.

  role\_list=opsusers,carecontent

Compare to 

  role\_list=opsusers,carenewfoo

and abruptly the whole thing comes crashing down.  

Also disconcerting was the process of removing a node.  At least a new node
would be monitorable withing a few minutes of coming up.  A terminated node is
still in the storeconfig DB, and there's no direct API for it, so you'd need
to script the SQL to delete the correct lines to effect a node removal.

That was about as far as I got before I realized I needed to take my hands off
the keyboard and step back.

TKTKTK How sensu and I found each other.  
Fortunately, there'd been some buzz on Twitter about Sensu, and over the
course of a weekend I became convinced that I needed to abandon Nagios, or any
other monolithic monitoring system, and try Sensu.

Sensu started as an internal project at Sonian, an archive-as-a-service
provider which runs on AWS.  Sean Porter and others had been using Nagios w/
Chef, but ran into many of the same convergence issues that I had described,
but also ran into scaling issues with Nagios's active check architecture.  So
Sean and a teammate, TKTK, wrote Sensu for internal use, and open-sourced it
in November of 2011.  Since then, it's seen a lot of uptake and has a very
active community.  Lots of help is available on IRC if you bother to stop in.

Backbone is the RabbitMQ message broker. 

Then we need at least one sensu-server, and typically on that box we'll run
Redis to provide persistence.  Sensu-server is written in Ruby, and can be
installed as a Gem, as an RPM, and soon as .deb.

All of the configuration is in JSON.  Here's a minimal configuration for a
sensu-server:

{
  "rabbitmq": {
    "host": "<%= rabbitmq_host %>",
    "port": <%= rabbitmq_port %>
  },
  "redis": {
    "host": "<%= redis_host %>",
    "port": <%= redis_port %>
  },
  "api": {
    "host": "<%= api_host %>",
    "port": <%= api_port %>
  },
}

What's missing from this is the definition of what checks to run.  Rather than
define the checks in a single massive JSON file, we can drop JSON snippets
into /etc/sensu/conf.d, like this:

{
  "checks": {
    "system_disks": {
      "handlers": ["irc", "mailer", "default" ],
      "notification": "System disk space is being exhausted",
      "command": "/etc/sensu/plugins/community/check-disk.rb -w 80 -c 90 -x tmpfs",
      "subscribers": [ "generic" ],
      "occurrences": 2,
      "interval": 300
    }
  }
}

OR

{
  "checks": {
    "careverge_api": {
      "handlers": ["irc", "default", "mailer" ],
      "notification": "Careverge API is not responding appropriately",
      "command": "/etc/sensu/plugins/local/check_cvapi.sh -S",
      "subscribers": [ "cvapi" ],
      "interval": 30,
      "refresh": 600
    }
  }
}

On the client side, we need only install sensu-client, and configure it.  The
configuration is pretty minimal: Specify the rabbitmq information and details
on this client:

{
  "rabbitmq": {
    "host": "<%= rabbitmq_host %>",
    "port": <%= rabbitmq_port %>
  },
  "client": {
    "name": "<%= sensu_hostname %>",
    "address": "<%= ipaddress %>",
    "subscriptions": [ "generic", "cvapi" ]
  }
}

All of the check details can go in conf.d/ again, and from a configuration
mgmt standpoint we have the added bonus that we can use the _exact same_
conf.d/ as we had on the server.  Now we can leverage the beauty of the
message queue 

* Sensu-server publishes a 'check-disk' request to the 'generic' channel every
  300s.
* Sensu-client how subscribe to 'generic' run the check and publish the
  results
* Sensu-server processes the results, and passes any failures to the handlers
* Likewise for the 30s interval checks of the API, but then it's only for the
  nodes subscribed to 'cvapi'.


Sensu works so well that I had to make sure that the check scripts were
installed before the sensu service. Not because there's any logical
dependency, but simply , otherwise the client would come up and
start acting on published requests for, say, 'check\_disks' and fail because
the 'check\_disk.rb' script wasn't there yet.

Handlers
--------

In order for anything useful to happen with a failed check result, we need
handlers to, say, notify us or even take action.  Let's take a look at IRC,
for example:

  irc.json

  irc.rb

That's about it.


API
---

Thin/Sinatra service running on port 4567

* Read and update key/values in Redis
* Publish check requests on RabbitMQ

For example:


KeepAlives
----------

Remember how we had issues with handling terminated nodes in Nagios?  In
Sensu, the clients will send keep-alives every 30s so if a sensu-client
service dies unexpectedly, or the node hosting it, we can know about it.

Upon an orderly system shutdown we can have a de-register itself through the
Sensu API.  Since we're currently on RightScale I've added this little script
to the Termination sequence on RightScale:


  #!/usr/bin/ruby

  config_file='/etc/sensu/client.json'
  json = File.read(config_file)

  client_name = JSON.parse(json)['client']['name']
  api_host    = JSON.parse(json)['api']['host']

  uri   = URI.parse("http://#{api_host}/client/#{client_name}")

  http  = Net::HTTP.new(uri.host, uri.port)
  http.request( Net::HTTP::Delete.new(uri.path) )


Dashboard
=========

One place where Sensu really shows its youth is in the interactive WebUI

(Three screenshots)

It is not yet PHB-compliant.

CheckPoint
==========

* RabbitMQ
* Redis 
* sensu-server: 
** Publishes check requests
** Pushes results to Handler
* sensu-client:
** Listens for check-requests on its subscriptions
** Runs check commands and publishes to MQ
* sensu-api
* sensu-dashboard

More Features
=============

But wait, there's more:

* Application integration
* Sensu and Graphite
* Standalone Checks
* Puppet integration
* Scheduling downtime
* Parameter passing
* Ideal monitoring system

Missing Features
================

* Reporting dashboard
* Service dependencies


Monitoring Requirements
-----------------------


- Integrate with Cloud Operations: CL
-- Instances should be automatically monitored
-- Cleanly decommissioned instances should go away
- Integrate with Configuration Management (Puppet or Chef): CM
-- Adding and removing checks should be facilitated by Puppet
- Provide 'sensible' alerting: AL
-- Partition urgency of alert by environment and/or time
- Support Developer Integration: DV
-- Can we code our apps to monitor themselves, or provide metrics
- Trending: TR
-- Gather data, present them and share them
- Comprehensive Checks: CH
-- Make it easy to monitor _anything_
-- If possible, implement _service dependencies_
- Available and Secure: AS
-- Also, we need to label our instances sensibly...
