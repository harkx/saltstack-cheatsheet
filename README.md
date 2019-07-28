# SaltStack Cheat Sheet

SaltStack Cheat Sheet .. My collection of often used commands on my Salt master.

This list is partly inspired by the fine lists on:
* http://www.xenuser.org/saltstack-cheat-sheet/
* https://github.com/saltstack/salt/wiki/Cheat-Sheet

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [SaltStack Cheat Sheet](#saltstack-cheat-sheet)
- [First things first : Documentation](#documentation)
  - [Documentation on the system](#documentation-on-the-system)
  - [Documentation on the web](#documentation-on-the-web)
- [Minions](#minions)
  - [Minion status](#minion-status)
  - [Target minion with state files](#target-minion-with-state-files)
  - [Grains](#grains)
- [Jobs in Salt](#jobs-in-salt)
- [Sysadmin specific](#sysadmin-specific)
  - [System and status](#system-and-status)
  - [Packages](#packages)
  - [Check status of a service and manipulate services](#check-status-of-a-service-and-manipulate-services)
  - [Network](#network)
- [Salt Cloud](#salt-cloud)

# Documentation
This is important because the help system is very good.

*notice: please do not target with \* if you have a large number of minions*

## Documentation on the system
```
### return execute module documents
salt '*' sys.doc         # output sys.doc (= all documentation)
salt '*' sys.doc pkg     # only sys.doc for pkg module
salt '*' sys.doc network # only sys.doc for network module
salt '*' sys.doc system  # only sys.doc for system module
salt '*' sys.doc status  # only sys.doc for status module

### execute module arguments
salt '*' sys.argspec pkg.install # show pkg.install arguments
salt '*' sys.argspec sys         # show all arguments under sys modeules
salt '*' sys.argspec 'sys.*'     # same with the above one

### state doc
salt '*' sys.state_doc                                    # show all states doc
salt '*' sys.state_doc service                            # show all service states doc
salt '*' sys.state_doc service.running                    # show service.running doc
salt '*' sys.state_doc service.running ipables.append     # show service.running and iptables.append docs
salt '*' sys.state_doc 'service.*' 'iptables.*'           # show all doc under service and iptables states

### state argument
salt '*' sys.state_argspec pkg.installed                  # show arguments of pkg.installed state
salt '*' sys.state_argspec file                           # show arguments under file states
salt '*' sys.state_argspec 'file.*'                       # same with above one

### sys.list.* states
salt '*' sys.list_functions                               # list all function
salt '*' sys.list_functions sys                           # list function under sys modules
salt '*' sys.list_functions 'sys.list_*'                  # list function under sys.list_

salt '*' sys.list_modules                                 # list all modules
salt '*' sys.list_renderers                               # list all renderers
salt '*' sys.list_returner_functions                      # list all returner functions
salt '*' sys.list_runners                                 # list all runners
salt '*' sys.list_state_modules                           # list all state modules

salt '*' sys.list_state_functions                                  # list all state functions
salt '*' sys.list_state_functions file                             # list all state functions under file state
salt '*' sys.list_state_functions pkg user                         # list all state function under pkg and user
```

## Documentation on the web
- SaltStack documentation: http://docs.saltstack.com/en/latest/
- Salt-Cloud: http://docs.saltstack.com/en/latest/topics/cloud/
- Jobs: http://docs.saltstack.com/en/latest/topics/jobs/

# Minions

## Minion status
You can also use several commands to check if minions are alive and kicking but I prefer manage.status/up/down.

```
salt-run manage.status  # What is the status of all my minions? (both up and down)
salt-run manage.up      # Any minions that are up?
salt-run manage.down    # Any minions that are down?
salt-run manage.alived  # Show all alive minions
salt-run manage.list_state
salt-run manage.list_not_state
salt-run manage.present
salt-run manage.not_present
salt-run manage.joined
salt-run manage.not_joined
salt-run manage.allowed
salt-run manage.not_allowed
salt-run manage.alived
salt-run manage.not_alived
salt-run manage.reaped
salt-run manage.not_reaped
salt-run manage.safe_accept my_minion
salt-run manage.safe_accept minion1,minion2 tgt_type=list
salt-run manage.versions
salt '*' test.version   # Display salt version
salt '*' test.ping      # Use test module to check if minion is up and responding.
                        # (Not an ICMP ping!)
```

## Target
### match minionid
```bash
salt '*' test.ping
salt \* test.ping

salt '*.example.net' test.version
salt '*.example.*' test.version
salt 'web?.example.net' test.version
salt 'web[1-5]' test.version
salt 'web[1,3]' test.version
salt 'web-[x-z]' test.version

salt -E 'web1-(prod|devel)' test.version
salt -L 'web1,web2,web3' test.version
```

### grains
```bash
salt -G 'os:Debian' test.ping
```

### pillar
```bash
salt -I 'key:value' test.ping
salt -I 'foo:bar:baz*' test.ping

```

### subnet/ip address
```bash
salt -S 192.168.40.20 test.version
salt -S 2001:db8::/64 test.version
salt -C 'S@10.0.0.0/24 and G@os:Debian' test.version
```

### node group
```bash
### need define group in /etc/salt/master first
salt -N group1 test.version
```

### compund match
```bash
#############################################
# G for grains glob, G@os:Debian
# E for PCRE expression, E@web\d+\.(dev|qa|prod)\.loc
# P for grains PCRE, P@os:(RedHat|Fedora|CentOS)
# L for lists of minions, L@minion1,minion2
# I for pillar glob, I@key:value
# J for pillar PCRE, J@key:^(value1|value2)$
# S for subnet/ip , S@192.168.1.0/24
# R for range cluster, R@foo.bar
# N for nodegroups, N@grup1
############################################
salt -C 'webserv* and G@os:Debian or E@web-dc1-srv.*' test.version
salt -C 'not web-dc1-srv' test.version
salt -C '* and not G@kernel:Darwin' test.version
salt -C '* and not web-dc1-srv' test.version
salt -C '( ms-1 or G@id:ms-3 ) and G@id:ms-3' test.version
```

## Grains
List all grains on all minions
```
salt '*' grains.ls
salt '*' grains.items
```

Look at a single grains item to list the values.
```
salt '*' grains.item os      # Show the value of the OS grain for every minion
salt '*' grains.item roles   # Show the value of the roles grain for every minion
salt '*' grains.get pkg:apache
salt '*' grains.get abc::def|ghi delimiter='|'
```

Manipulate grains.
```
salt 'minion1' grains.setval mygrain True  # Set mygrain to True (create if it doesn't exist yet)
salt 'minion1' grains.delval mygrain       # Delete the value of the grain
```

## Pillars
```bash
salt '*' pillar.items
salt '*' pillar.get pkg:apache
salt '*' pillar.get abc::def|ghi delimiter='|'
salt '*' pillar.filter_by '{salt*: got some salt, default: salt is not here}' role
salt '*' pillar.filter_by '{web: Serve it up, db: I query, default: x_x}' role
salt '*' pillar.get pkg:apache
salt '*' pillar.get abc::def|ghi delimiter='|'
salt '*' pillar.item foo
salt '*' pillar.item foo:bar
salt '*' pillar.item foo bar baz
salt '*' pillar.keys web:sites
salt '*' pillar.ls
salt '*' pillar.obfuscate
salt '*' pillar.raw
salt '*' pillar.raw key='roles'
```

## events
```bash
salt '*' event.fire '{"data":"my event data"}' 'tag'
salt '*' event.fire_master '{"data":"my event data"}' 'tag'
salt-call event.send myco/mytag foo=Foo bar=Bar
salt-run state.event pretty=True
```

## schedule
```bash
salt '*' schedule.add job1 function='test.ping' seconds=3600
salt '*' schedule.list
salt '*' schedule.list show_all=True
salt '*' schedule.delete job1
salt '*' schedule.disable
salt '*' schedule.disable_job job1
salt '*' schedule.enable
salt '*' schedule.enable_job job1
```

## miscs
```bash
salt-run reactor.list
salt-run reactor.add 'salt/cloud/*/destroyed' reactors='/srv/reactor/destroy/*.sls'
salt-run reactor.delete 'salt/cloud/*/destroyed'
salt-run reactor.is_leader
salt-run reactor.set_leader True

salt-run saltutil.sync_all
salt-run saltutil.sync_all extmod_whitelist={'runners': ['custom_runner'], 'grains': []}
salt-run saltutil.sync_modules
salt-run saltutil.sync_states
salt-run saltutil.sync_grains

salt-run mine.get '*' network.interfaces
salt-run mine.update '*'
salt-run mine.update 'juniper-edges' tgt_type='nodegroup'

salt-run http.query http://somelink.com/
salt-run http.query http://somelink.com/ method=POST

salt-run survey.hash "*" test.ping
salt-run survey.hash "*" file.get_hash /etc/salt/minion survey_sort=up
salt-run survey.diff [survey_sort=up/down] <target>
salt-run survey.diff survey_sort=up "*" cp.get_file_str file:///etc/hosts

salt-run cache.grains '*'
salt-run cache.pillar
salt-run cache.mine
salt-run cache.clear_pillar
salt-run cache.clear_grains
salt-run cache.clear_mine
salt-run cache.clear_mine_func tgt='*' clear_mine_func_flag='network.interfaces'
salt-run cache.clear_all

salt '*' state.apply
salt '*' state.apply stuff pillar='{"foo": "bar"}'
salt '*' state.apply exclude=bar,baz
salt '*' state.apply stuff sync_mods=all
salt '*' state.check_request
salt '*' state.clear_cache
salt '*' state.clear_request
salt '*' state.disable highstate
salt '*' state.disable highstate,test.succeed_without_changes
salt '*' state.disable bind.config
salt '*' state.enable highstate
salt '*' state.enable test.succeed_without_changes
salt '*' state.disable bind.config
salt '*' state.high '{"vim": {"pkg": ["installed"]}}'
salt '*' state.highstate
salt '*' state.highstate whitelist=sls1_to_run,sls2_to_run
salt '*' state.highstate exclude=sls_to_exclude
salt '*' state.list_disabled
salt '*' state.low '{"state": "pkg", "fun": "installed", "name": "vi"}'
salt '*' state.request
salt '*' state.run_request
salt '*' state.running
salt '*' state.show_highstate
salt '*' state.show_low_sls foo
salt '*' state.show_lowstate
salt '*' state.show_state_usage
salt '*' state.show_top
```

# Jobs in Salt
Some jobs operations that are often used. (http://docs.saltstack.com/en/latest/topics/jobs/)
```
salt-run jobs.active      # get list of active jobs
salt-run jobs.list_jobs   # get list of historic jobs
salt-run jobs.lookup_jid <job id number>  # get details of this specific job
salt-run jobs.print_job 20130916125524463507
salt-run jobs.exit_success 20160520145827701627
salt-run jobs.last_run
salt-run jobs.last_run target=nodename
salt-run jobs.last_run function='cmd.run'
```

# Sysadmin specific
Some stuff that is specifically of interest for sysadmins.

## System and status
```
salt 'minion-x-*' system.reboot  # Let's reboot all the minions that match minion-x-*
salt '*' status.uptime           # Get the uptime of all our minions
```

## Packages
```
salt '*' pkg.list_upgrades             # get a list of packages that need to be upgrade
salt '*' pkg.upgrade                   # Upgrades all packages via apt-get dist-upgrade (or similar)

salt '*' pkg.version bash              # get current version of the bash package
salt '*' pkg.install bash              # install or upgrade bash package
salt '*' pkg.install bash refresh=True # install or upgrade bash package but
                                      # refresh the package database before installing.
```

## Check status of a service and manipulate services
```
salt '*' service.status <service name>
salt '*' service.available <service name>
salt '*' service.start <service name>
salt '*' service.restart <service name>
salt '*' service.stop <service name>
```

## Network

Do some network stuff on your minions.

```
salt 'minion1' network.ip_addrs          # Get IP of your minion
salt 'minion1' network.ping <hostname>   # Ping a host from your minion
salt 'minion1' network.traceroute <hostname>   # Traceroute a host from your minion
salt 'minion1' network.get_hostname      # Get hostname
salt 'minion1' network.mod_hostname      # Modify hostname
```

# Salt Cloud
Salt Cloud is used to provision virtual machines in the cloud. (surprise!) (http://docs.saltstack.com/en/latest/topics/cloud/)

```
salt-cloud -p profile_do my-vm-name -l debug  # Provision using profile_do as profile
                                              # and my-vm-name as the virtual machine name while
                                              # using the debug option.
salt-cloud -d my-vm-name                      # destroy the my-vm-name virtual machine.
salt-cloud -u                                 # Update salt-bootstrap to latest develop version on GitHub.
```

