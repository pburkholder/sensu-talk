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
software.  Wherever we sit in relation to this mess of running stuff, we'll
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

The problem

Let's dispense with the high-level view of the space and dig into the
particulars of a problem our team has been tackling as way of illustrating how
differing monitoring frameworks fit with a high-churn cloud-based
infrastructure.

At Audax Health we run health consume social network called Careverge, and the
core architecture should be not unfamiliar to most of you.  Web traffic to 
www.careverge.com will be 

The way I've generally tackled this problem over the last few gigs has been
with the Nagios monitoring framework because, despite the age of some of the
code, Ethan Galstad has gotten a lot of things right that I've seen utterly
absent from recent vintage enterprise products (don't go into MySQL monitor)


The way I've met the first two needs in the 

