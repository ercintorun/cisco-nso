To use NSO Python API, you typically run the Python application on the same machine as NSO, as it uses TCP socket IPC to communicate with the NSO daemon

,It is incumbent on the NSO Python developer to have a working knowledge of the NSO and NED YANG model or reference the WebUI to see the YANG types needed for a desired Python activity
  
#Python Examples 
## CDP Audit
  
    -----nso-test.py
    import ncs
    with ncs.maapi.single_read_trans("admin", "python", groups=["ncsadmin"]) as t:
        root = ncs.maagic.get_root(t)
        device = root.devices.device["device-name-here"]
        cdp_result = device.config.ios__cdp.run
        print(
            "For Device {}, CDP being enabled is {}".format(device.name, cdp_result)
        ) 
    -----
    developer:src > python3 nso-test.py 
    For Device netsim-ios, CDP being enabled is True
    
## Interface Audit

    -----nso-test.py
    import ncs
    with ncs.maapi.single_read_trans("admin", "python", groups=["ncsadmin"]) as t:
        root = ncs.maagic.get_root(t)
        device = root.devices.device["device-name"]
        for interface in device.config.ios__interface:
            for if_type in device.config.ios__interface[interface]:
                if hasattr(if_type, "name"):
                    print(
                        f'Device {device.name}, Interface {if_type} {if_type.name}'
                    )
    -----
    
    developer:src > python3 nso-test.py 
    Device netsim-ios, Interface Loopback 0
    Device netsim-ios, Interface FastEthernet 0/0
    Device netsim-ios, Interface FastEthernet 1/0
    Device netsim-ios, Interface FastEthernet 1/1
