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

## Manipulating Device Data 
    
