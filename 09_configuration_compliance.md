This use case has two parts.

Building a device template that contains the desired configuration to test.
Building a compliance report to check the configuration of a group of devices against a template.

## Building the device template

    ncs_cli -C -u admin
    config
    devices template COMPLIANCE-CHECK
    ned-id cisco-ios-cli-6.91
    config
    ip name-server name-server-list 208.67.222.222
    ip name-server name-server-list 208.67.220.220
    exit
    service timestamps log datetime localtime show-timezone year
    logging host ipv4 10.225.1.11
    exit
    ntp server peer-list 10.225.1.11
    exit
    exit
    exit
    !
    ned-id cisco-nx-cli-5.23
    config
    ip name-server servers 208.67.222.222
    ip name-server servers 208.67.220.220
    logging timestamp milliseconds
    logging server 10.225.1.11 level 5
    exit
    ntp server 10.225.1.11
    exit
    exit
    exit
    !
    ned-id cisco-asa-cli-6.6
    config
    dns domain-lookup mgmt
    dns server-group DefaultDNS
    name-server 208.67.222.222
    name-server 208.67.220.220
    exit
    logging timestamp
    logging host mgmt 10.225.1.11
    exit
    ntp server 10.225.1.11
    exit
    exit
    exit

    top
    show configuration
    commit

## Building the compliance report

* Results can come in in HTML, Bash, or XML format. XML is the default.
* Because the results are likely very "verbose" and not something the engineer wants to see only at the CLI, NSO saves the results to a file located in the folder state/compliance-reports.
* NSO runs a web server so these reports are available by navigating to the given URL, or you can find them by navigating the folders on your server.

    compliance reports report COMPLIANCE-CHECK
    compare-template COMPLIANCE-CHECK ALL
    commit
    end
    #All is a device group here.

From the "enable mode" of NSO

    compliance reports report COMPLIANCE-CHECK run

Output: 

    admin@ncs# compliance reports report COMPLIANCE-CHECK run
    id 1
    compliance-status violations
    info Checking 8 devices and no services
    location http://localhost:8080/compliance-reports/report_1_admin_1_2020-3-24T9:48:41:0.xml
    admin@ncs#

Generate different type of reports: 

    compliance reports report COMPLIANCE-CHECK run outformat text
    compliance reports report COMPLIANCE-CHECK run outformat html

Note: Even though NSO evaluates different CLI device types (IOS, NX, ASA), NSO standardizes the "native" configuration into a uniform data model, in Junos curly bracket style, when showing the diff.

## Resolving Compliance Problems

    ncs_cli -C -u admin
    config
    
    devices device-group ALL apply-template template-name COMPLIANCE-CHECK
    show configuration
    commit dry-run outformat native
    commit
    end
    compliance reports report COMPLIANCE-CHECK run outformat text
