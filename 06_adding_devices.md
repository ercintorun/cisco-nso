## Creating Authgroups

devices authgroups group labadmin
default-map remote-name cisco
default-map remote-password cisco
default-map remote-secondary-password cisco

You can create as many authgroups as needed for the different variations of authentication used across your network. An authgroup can use SNMP or certificates in addition to traditional username/passwords. NSO supports powerful methods of RBAC for identifying and using different authentication and authorization levels depending on the user who is logged into NSO


## Adding Devices to Cisco NSO 

There are 3 methods to add devices: 
* You can manually type the commands for each device
* You can use an API to add devices to the device list
* You can query the production system install NSO server that has the full device list already and save that output to a file. You can then use the load merge command from NSO to read the file into NSO and stage it for committing.


### Manual method
    devices device edge-sw01
    address 10.10.20.172
    authgroup labadmin
    device-type cli ned-id cisco-ios-cli-6.91
    device-type cli protocol telnet
    ssh host-key-verification none
    commit

NSO's default mode for devices is a locked state that prevents NSO from manipulating a device before the network admin is ready. You can use this state to disable a device that is currently in maintenance to prevent NSO from acting on it.
    state admin-state unlocked
    commit

### Get Config From Production NSO 
To get configuration from production NSO server: 
    show running-config devices device | de-select config

Copy the output of that command (except for the prompts developer@ncs#) and save it into a file called nso-device-config.txt
Close the terminal session to the production system install. Using your local install NSO on the devbox

    ncs_cli -C -u admin
    conf
    load merge /home/developer/nso-device-config.txt
    commit
    end
    
## Grouping Devices Together
You create device groups to organize your devices into logical groups. Then you can apply similar configuration or take similar actions on a group.

    devices device-group IOS-DEVICES
    device-name internet-rtr01
    device-name dist-rtr01
    device-name dist-rtr02
    commit
    end
    devices device-group IOS-DEVICES check-sync
    
