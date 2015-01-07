# SaltStack Cheat Sheet

SaltStack Cheat Sheet .. My collection of often used commands on my Salt master.

This list is partly inspired by the fine lists on:
* http://www.xenuser.org/saltstack-cheat-sheet/
* https://github.com/saltstack/salt/wiki/Cheat-Sheet

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [SaltStack Cheat Sheet](#SaltStack-Cheat-Sheet)
- [Minion status](#)
- [Jobs in Salt](#)
- [Sysadmin specific](#)
	- [System and status](#)
	- [Packages](#)
	- [Target minion with state files](#)
	- [Check status of a service and manipulate services](#)
- [Salt Cloud](#)
- [Grains](#)
- [Documentation](#)
	- [Documentation on the system](#)
	- [Documentation on the web](#)


# Minion status

You can also use several commands to check if minions are alive and kicking but I prefer manage.status/up/down.

```
salt-run manage.status  # What is the status of all my minions? (both up and down)
salt-run manage.up      # Any minions that are up?
salt-run manage.down    # Any minions that are down?
salt '*' test.ping       # Use test module to check if minion is up and responding.
                        # (Not an ICMP ping!)
```

# Jobs in Salt

Some jobs operations that are often used. (http://docs.saltstack.com/en/latest/topics/jobs/)
```
salt-run jobs.active      # get list of active jobs
salt-run jobs.list_jobs   # get list of historic jobs
salt-run jobs.lookup_jid <job id number>  # get details of this specific job
```

# Sysadmin specific

Some stuff that is specifically of interest for sysadmins.

## System and status

```
salt '*' system.reboot
salt '*' status.uptime
```

## Packages

```
salt '*' pkg.list_upgrades             # get a list of packages that need to be upgrade

salt '*' pkg.version bash              # get current version of the bash package
salt '*' pkg.install bash              # install or upgrade bash package
salt '*' pkg.install bash refresh=True # install or upgrade bash package but
                                      # refresh the package database before installing.
```

## Target minion with state files

Apply a specific state file to a (group of..) minion(s). Do not use the .sls extension. (just like in the state files!)

```
salt '*' state.sls mystatefile           # mystatefile.sls will be applied to *
salt 'minion1' state.sls prod.somefile  # prod/somefile.sls will be applied to minion1
```

## Check status of a service and manipulate services

```
salt '*' service.status <service name>
salt '*' service.available <service name>
salt '*' service.start <service name>
salt '*' service.restart <service name>
salt '*' service.stop <service name>
```

# Salt Cloud

Salt Cloud is used to provision virtual machines in the cloud. (surprise!) (http://docs.saltstack.com/en/latest/topics/cloud/)

```salt-cloud -p profile_do my-vm-name -l debug``` - Provision using profile_do as profile and my-vm-name as the virtual machine name while using the debug option.

```salt-cloud -d my-vm-name``` - destroy the my-vm-name virtual machine.

```salt-cloud -u``` - Update salt-bootstrap to the latest develop version on GitHub.

# Grains

List all grains on all minions
```
salt '*' grains.ls
```

Look at a single grains item to list the values.
```
salt '*' grains.item os      # Show the value of the OS grain for every minion
salt '*' grains.item roles   # Show the value of the roles grain for every minion
```

# Documentation

## Documentation on the system

```
salt '*' sys.doc         # output sys.doc (= all documentation)
salt '*' sys.doc pkg     # only sys.doc for pkg module
salt '*' sys.doc network # only sys.doc for network module
salt '*' sys.doc system  # only sys.doc for system module
salt '*' sys.doc status  # only sys.doc for status module
```

## Documentation on the web

- SaltStack documentation: http://docs.saltstack.com/en/latest/
- Salt-Cloud: http://docs.saltstack.com/en/latest/topics/cloud/
- Jobs: http://docs.saltstack.com/en/latest/topics/jobs/
