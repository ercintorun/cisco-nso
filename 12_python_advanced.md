You can find the NSO Python API docs [on DevNet here](https://developer.cisco.com/docs/nso/api/#!ncs)

The typical Flow of using the NSO Maagic Python API is as follows:
1. Create a transaction
2. Access device information
3. Manipulate data
4. Apply configurations
5. Close the transaction

## Create Transaction 
There are several options for opening a connection to NSO through the API. The most common ones to use are as follows:
 single_read_trans
 single_write_trans
Most Python scripts are going to have a variation of the following statement:
    
    with ncs.maapi.single_write_trans('admin', 'python', groups=['ncsadmin']) as t:

This statement opens a write transaction to NSO (like entering config mode on CLI); if you want to limit the access to read-only, you can replace single_write_trans with single_read_trans. While using single_write_trans, allow reading and writing data.
The admin part tells NSO which user for logging purposes will be accessing NSO. 
The groups parameter is not needed on a local install, but often system installs have a ncsadmin Linux group set up.

## Access device Information 

    python3
    import ncs
    with ncs.maapi.single_read_trans('admin', 'python', groups=['ncsadmin']) as t:
        root = ncs.maagic.get_root(t)
        device_object  = root.devices.device["netsim-ios"]
        print(device_object.name)

root.devices.device references all the devices in the NSO CDB. Even though the name says '.device' it is a YANG list, not one device.
You can either reference the NSO YANG model, the NSO GUI, or use the print dir() option in Python to determine available attribute

Below code snippet prints out the available object methods associated with the device object:

    ----code snipper
    python3
    import ncs
    
    with ncs.maapi.single_read_trans('admin', 'python', groups=['ncsadmin']) as t:
        root = ncs.maagic.get_root(t)
        device_object  = root.devices.device["netsim-ios"]
       print(dir(device_object))
    ----output
     developer:src > python3
    Python 3.11.2 (main, Jul 18 2023, 04:42:25) [GCC 8.5.0 20210514 (Red Hat 8.5.0-18)] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> >>> >>> ... ... ... 
    ... 
    ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__self__', '__self_class__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__thisclass__', 'active_settings', 'add_capability', 'address', 'address_choice', 'al__alarm_summary', 'apply_template', 'authgroup', 'capability', 'check_sync', 'check_yang_modules', 'choice_lsa', 'commit_queue', 'compare_config', 'config', 'connect', 'connect_retries', 'connect_timeout', 'copy_capabilities', 'delete_config', 'description', 'device_profile', 'device_type', 'disconnect', 'find_capabilities', 'instantiate_from_other_device', 'last_changed', 'live_status', 'live_status_protocol', 'load_native_config', 'local_user', 'location', 'lsa', 'lsa_remote_node', 'migrate', 'module', 'name', 'ned_keep_alive', 'ned_settings', 'netconf_notifications', 'no_lsa', 'no_overwrite', 'no_wait_for_lock', 'notifications', 'out_of_sync_commit_behaviour', 'ping', 'platform', 'port', 'read_timeout', 'rpc', 'scp_from', 'scp_to', 'service_list', 'session_limits', 'session_pool', 'snmp_notification_address', 'source', 'ssh', 'ssh_algorithms', 'ssh_keep_alive', 'state', 'sync_from', 'sync_to', 'trace', 'trace_output', 'use_lsa', 'wait_for_lock', 'wait_for_lock_choice', 'write_timeout']
    >>> 

# Manipulating Device Data 
## Manipulate Data - CDP 
Use the NSO CLI to find the path to the device configuration and see the namespace ios::

    ncs_cli -C -u admin
    paginate false
    show running-config devices device netsim-ios config | display xpath
    exit

    ----output
    developer:~ > ncs_cli -C -u admin
    admin@ncs# paginate false
    admin@ncs# show running-config devices device netsim-ios config | display xpath
    /devices/device[name='netsim-ios']/config/ios:tailfned/device netsim
    /devices/device[name='netsim-ios']/config/ios:tailfned/police cirmode
    /devices/device[name='netsim-ios']/config/ios:ip/source-route true
    /devices/device[name='netsim-ios']/config/ios:ip/vrf[name='my-forward']/bgp/next-hop/Loopback 1
    /devices/device[name='netsim-ios']/config/ios:ip/http/server false
    /devices/device[name='netsim-ios']/config/ios:ip/http/secure-server false
    /devices/device[name='netsim-ios']/config/ios:ip/community-list/number-standard[no='1']/permit
    /devices/device[name='netsim-ios']/config/ios:ip/community-list/number-standard[no='2']/deny
    /devices/device[name='netsim-ios']/config/ios:ip/community-list/standard[name='s']/permit
    /devices/device[name='netsim-ios']/config/ios:class-map[name='a']/prematch match-all
    /devices/device[name='netsim-ios']/config/ios:class-map[name='cmap1']/prematch match-all
    /devices/device[name='netsim-ios']/config/ios:class-map[name='cmap1']/match/mpls/experimental/topmost [ 1 ]
    /devices/device[name='netsim-ios']/config/ios:class-map[name='cmap1']/match/packet/length/max 255
    /devices/device[name='netsim-ios']/config/ios:class-map[name='cmap1']/match/packet/length/min 2
    /devices/device[name='netsim-ios']/config/ios:class-map[name='cmap1']/match/qos-group [ 1 ]
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='a']
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/drop
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/estimate/bandwidth
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/estimate/bandwidth/delay-one-in/doi 500
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/estimate/bandwidth/delay-one-in/milliseconds 100
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/priority
    /devices/device[name='netsim-ios']/config/ios:policy-map[name='map1']/class[name='c1']/priority/percent 33
    /devices/device[name='netsim-ios']/config/ios:interface/Loopback[name='0']
    /devices/device[name='netsim-ios']/config/ios:interface/FastEthernet[name='0/0']
    /devices/device[name='netsim-ios']/config/ios:interface/FastEthernet[name='1/0']
    /devices/device[name='netsim-ios']/config/ios:interface/FastEthernet[name='1/1']
    /devices/device[name='netsim-ios']/config/ios:spanning-tree/optimize/bpdu/transmission false
    /devices/device[name='netsim-ios']/config/ios:router/bgp[as-no='64512']/aggregate-address/address 10.10.10.1
    /devices/device[name='netsim-ios']/config/ios:router/bgp[as-no='64512']/aggregate-address/mask 255.255.255.251
    admin@ncs# exit

The path within the application would be:
Devices - Device - DEVICENAME - CONFIG - IOS:CDP - RUN, with a value set to true.

    python3
    import ncs
    with ncs.maapi.single_write_trans('admin', 'python', groups=['ncsadmin']) as t:
            root = ncs.maagic.get_root(t)
            device_object = root.devices.device["netsim-ios"].config
            device_object.ios__cdp.run = True
            t.apply()

When applying the change, NSO checks to see if the change needs to be done by first checking if the
device is in sync, and if it is, checking to see if the change even needs to be made. If the change needs to be made, NSO sends the necessary commands and records the rollback for later use.

## Manipulate Data - Adding/Removing Loopback

Add:

    python3
    import ncs
    with ncs.maapi.single_write_trans('admin', 'python', groups=['ncsadmin']) as t:
        root = ncs.maagic.get_root(t)
        device_cdb = root.devices.device["netsim-ios"]
        device_cdb.config.ios__interface["Loopback"].create("1337")
        device_cdb.config.ios__interface.Loopback["1337"].ip.address.primary.address = "192.168.1.1"
        device_cdb.config.ios__interface.Loopback["1337"].ip.address.primary.mask = "255.255.255.252"
        t.apply()

Remove:

    python3
    import ncs
    with ncs.maapi.single_write_trans('admin', 'python', groups=['ncsadmin']) as t:
        root = ncs.maagic.get_root(t)
        device_cdb = root.devices.device["netsim-ios"]
        del device_cdb.config.ios__interface.Loopback["1337"]
        t.apply()
