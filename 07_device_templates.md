For multi-device configuration,  the simplest way to get started is using device templates.

    ncs_cli -C -u admin
    config
    devices template SET-DNS-SERVER
    ! IOS TEMPLATE
    ned-id cisco-ios-cli-6.91
    config
    ip name-server name-server-list 208.67.222.222
    ip name-server name-server-list 208.67.220.220
    exit
    exit
    exit
    
    ! ASA TEMPLATE
    ! 3 exits to return back to the 'config-template-SET-DNS-SERVER' context
    ned-id cisco-asa-cli-6.6
    config
    dns domain-lookup mgmt
    dns server-group DefaultDNS
    name-server 208.67.220.220
    name-server 208.67.222.222
    exit
    exit
    exit
    
    ! IOS-XR TEMPLATE
    ned-id cisco-iosxr-cli-7.45
    config
    domain name-server 208.67.222.222
    exit
    domain name-server 208.67.220.220

Let's see what this template looks like

    top
    show configuration 
    commit

Apply template
    
    devices device-group ALL apply-template template-name SET-DNS-SERVER
    commit dry-run outformat native.
    commit
    
