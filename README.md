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






