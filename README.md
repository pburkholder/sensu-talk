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

Audax only: Replace RightScale: RS




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




Sensu works so well that I had to make sure that the check scripts were
installed before the sensu service. Not because there's any logical
dependency, but simply , otherwise the client would come up and
start acting on published requests for, say, 'check\_disks' and fail because
the 'check\_disk.rb' script wasn't there yet.
