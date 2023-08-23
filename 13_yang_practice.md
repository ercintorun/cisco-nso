https://developer.cisco.com/learning/tracks/get_started_with_nso/nso-labs/yang-nso-101/introduction/ 
## Use RESTCONF to read interface information

We need to use postman for Restconf tests to see YANG structured replies etc: 
https://10.10.20.175/restconf/data/ietf-interfaces:interfaces/interface=GigabitEthernet1
Also you need to configure basic-aut in postman 

XML Response: 

    <interface xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces"  xmlns:if="urn:ietf:params:xml:ns:yang:ietf-interfaces">
        <name>GigabitEthernet1</name>
        <description>to port6.sandbox-backend</description>
        <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
        <enabled>true</enabled>
        <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
            <address>
                <ip>10.10.20.175</ip>
                <netmask>255.255.255.0</netmask>
            </address>
        </ipv4>
        <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
      </ipv6>
    </interface>

On the Headers tab, add two new keys: Accept and Content-Type.
Set the value for both to application/yang-data+json

Json response (in yang data structure)

    {
        "ietf-interfaces:interface": {
            "name": "GigabitEthernet1",
            "description": "to port6.sandbox-backend",
            "type": "iana-if-type:ethernetCsmacd",
            "enabled": true,
            "ietf-ip:ipv4": {
                "address": [
                    {
                        "ip": "10.10.20.175",
                        "netmask": "255.255.255.0"
                    }
                ]
            },
            "ietf-ip:ipv6": {}
        }
    }

## Create a loopback interface

POST https://10.10.20.175/restconf/data/ietf-interfaces:interfaces/
Accept: application/yang-data+json
Content-Type: application/yang-data+json
Auth: Basic Auth 
username: ..
password: ..
body type: raw 


    POST https://10.10.20.175/restconf/data/ietf-interfaces:interfaces/
    Accept: application/yang-data+json
    Content-Type: application/yang-data+json
    
    {
      "ietf-interfaces:interface": {
        "name": "Loopback101",
        "description": "Configured by RESTCONF",
        "type": "iana-if-type:softwareLoopback",
        "enabled": true
      }
    }

You should receive a 201 Created response. This means that you successfully created a Loopback100 interface, configured the description, and enabled the interface
