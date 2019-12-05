# Cloudforms Troubleshooting 101

## evm.log:

`evm.log` is the main log file of CloudForms.  It's the log to follow for most of the basic problems with CloudForms.

First check to see what roles are on which systems in their environment:

`# grep local= evm.log`

Example:
```
[----] I, [2019-12-05T13:44:53.486372 #25740:bf0f68]  INFO -- : MiqServer: local=N, master=N, status= started, id=00004, pid=08944, guid=f1b6476f-b3e4-4ef0-a9e3-8f605b39d738, name=EVM, zone=RHV, hostname=cfme-rhv.example.com, ipaddress=10.0.0.1, version=5.10.12.3, build=20191029180220_f883bde, active roles=automate:database_operations:ems_inventory:ems_operations:event:reporting:smartstate:user_interface:web_services:websocket
[----] I, [2019-12-05T13:44:53.490625 #25740:bf0f68]  INFO -- : MiqServer: local=Y, master=N, status= started, id=00001, pid=25740, guid=d8658f16-0ba7-44b8-8f97-69501e6ae999, name=EVM, zone=default, hostname=cfme.example.com, ipaddress=10.0.0.2, version=5.10.12.3, build=20191029180220_f883bde, active roles=automate:database_operations:database_owner:embedded_ansible:ems_inventory:ems_operations:event:notifier:reporting:user_interface:web_services:websocket
[----] I, [2019-12-05T13:44:53.494736 #25740:bf0f68]  INFO -- : MiqServer: local=N, master=Y, status= started, id=00003, pid=32399, guid=a9cec71e-eda5-4b0d-90f7-24dd03e6b208, name=EVM, zone=vCenter Zone, hostname=cfme-vcenter1.example.com, ipaddress=10.0.0.3, version=5.10.12.3, build=20191029180220_f883bde, active roles=automate:database_operations:ems_inventory:ems_metrics_collector:ems_metrics_coordinator:ems_metrics_processor:ems_operations:event:reporting:scheduler:smartproxy:smartstate:web_services:websocket
[----] I, [2019-12-05T13:44:53.498199 #25740:bf0f68]  INFO -- : MiqServer: local=N, master=N, status= started, id=00002, pid=31673, guid=0613c826-1240-4327-bf94-f452e55acc93, name=EVM, zone=vCenter Zone, hostname=cfme-vcenter2.example.com, ipaddress=10.0.0.4, version=5.10.12.3, build=20191029180220_f883bde, active roles=automate:database_operations:ems_metrics_collector:ems_metrics_processor:ems_operations:reporting:smartproxy:smartstate:web_services:websocket
```

How to read these lines:

- local = Do the logs belong to the appliance?  N = no, Y = Yes
- master = Is the appliance the master appliance?
- name = name of the appliance
- zone = zone of the appliance
- roles = Roles enabled on the appliance

Then determine if they're the relevant logs, grep out all the errors and warnings.  In general, once a relevant error log line is found, do a search to see if a bug is already open for it:

`# grep -E ' E, | W, ' evm.log`

Then I get the timestamp for the relevant error/warning and then do a less on the log to get the full stack trace.

## automation.log

This is the main log for provisioning issues and issues dealing with automate domains.  This is the log you will want to search when customer reports provisioning issues.  Sometimes it will be due to custom code, and Red Hat offical support policy is that Red Hat doesn't troubleshoot customer's custom code they have created.

Determine if there is an automate role on the specific appliance logs you have, from there, do a similar grep for the `evm.log`:

`# grep -E ' E, | W, ' automation.log`

Again, once you find relevant error, copy the time stamp, do a less, and then search for the timestamp to get a full stacktrace.

## production.log

This is a log to see errors that happen in the UI.  This time, I like to do an additional grep for `FATAL`:

`# grep -Ei 'fatal|error|warn' production.log`

## top_output.log

If you see any memory exceeded errors or refresh problems, it's good to check this log in order to see if the appliance is overburdened.  This can happen if there are too many roles enabled on the appliance.
Something that can remedy this is to either provision more appliances, adjust memory settings for workers, or more evenly distribute roles among appliances.


## Provider specific logs

If it's a provider specific issue, these are the logs you need to look into, here are the more common providers:

- `aws.log` - AWS
- `azure.log` - Azure
- `fog.log` - Openstack
- `kubernetes.log` - Openshift
- `rhevm.log` - Red Hat Enterprise Virtualization Manager
- `vim.log` - vCenter
