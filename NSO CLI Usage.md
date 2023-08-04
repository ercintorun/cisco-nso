# NSO CLI Usage

The NSO CLI comes in both a "Juniper-style" and a "Cisco-style" flavor. Start exploring by:

    ncs_cli -u admin -C

Synchronize all the configurations in the network into the NSO CDB (Configuration Database)

    devices sync-from

NSO keeps a copy of the device configuration in CDB, and you can quickly check if devices are in sync or not:

    devices device ios1 check-sync
