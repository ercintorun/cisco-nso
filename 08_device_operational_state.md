## Basic platform information
Basic platform information stored in the NSO CDB. As part of the initial connection to a device, NSO queries the device for its model, serial, version, and so on.

    ncs_cli -C -u admin
    show devices device dist-sw01 platform
    show devices device dist-sw01 platform serial-number
    show devices device * platform serial-number
    show devices device * platform version

Note: Old versions of the IOS-XR platform may display N/A in the serial number.

## Live status

    show devices device dist-sw01 live-status port-channel
    show devices device dist-sw01 live-status ip route

When something is missing from a NED, you can open a support ticket with the Cisco NSO team explaining what is missing, and the team can update NEDs to support what is needed. These updates often only take a few days for vetting, testing, and implementing by Cisco, and then making the updated NEDs available.
However, for immediate needs, live-status supports sending arbitrary commands to a network device and returning the result. 

    devices device dist-sw01 live-status exec show license usage
    devices device dist* live-status exec show license usage | save /home/developer/nexus-license-usage.txt

You can't use live-status with device-groups because a group of devices could leverage different NEDs.
