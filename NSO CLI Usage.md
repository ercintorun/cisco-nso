# NSO CLI Usage

The NSO CLI comes in both a "Juniper-style" and a "Cisco-style" flavor. Start exploring by:

    ncs_cli -u admin -C

Synchronize all the configurations in the network into the NSO CDB (Configuration Database)

    devices sync-from

NSO keeps a copy of the device configuration in CDB, and you can quickly check if devices are in sync or not:

    devices device ios1 check-sync

## Configuring The Network 
Go into configuration mode using the command

    config

### Examining Configuration
You can examine the configuration for the device ios1:

    show full-configuration devices device ios1 config | nomore
    show full-configuration devices device ios1 config | include bgp

### Changing Configuration
Change  context to be of the device we want to modify:

    devices device ios0 config
    ios:hostname nso.cisco.com

This does not immediately push the configuration to the device. Instead, it adds the change to the current transaction. Return to the top-level context, and examine the changes 
    top
    show configuration

Change multiple devices simultaneously by setting the enable password for all our devices:
    devices device * config ios:enable password magic

When done editing, you have two main alternatives: revert to throw our changes away or commit to push them to the network. A dry-run shows which configuration commands would be executed for each device you modified. Since NSO is a multi-vendor platform, appearance may differ between device types.

    commit dry-run outformat native
    commit
    end
