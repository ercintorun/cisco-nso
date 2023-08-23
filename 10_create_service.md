NSO service is a method to abstract and automate a task you want to de repeteadly. 

# Creating the Service package
Service packages are a collection of structured files and folders that NSO loads in to the application to extend NSO with new functionality

    cd nso-instance/packages/
    ls
    ncs-make-package -h
    ncs-make-package --service-skeleton template loopback-service
    tree loopback-service/

The key files that you are editing are loopback-service/src/yang/loopback-service.yang and loopback-service/templates/loopback-service-template.xml. Services can also coding logic, but in this simple example, you only use the most basic features.

     (py3venv) [developer@devbox packages]$ tree loopback-service/
     loopback-service/
     ├── package-meta-data.xml
     ├── src
     │   ├── Makefile
     │   └── yang
     │       └── loopback-service.yang
     ├── templates
     │   └── loopback-service-template.xml
     └── test
         ├── Makefile
         └── internal
            ├── Makefile
            └── lux
                 ├── Makefile
                 └── basic
                 ├── Makefile
                 └── run.lux

View the contents of loopback-service/src/yang/loopback-service.yang and loopback-service/templates/loopback-service-template.xml.

    cat loopback-service/src/yang/loopback-service.yang
    cat loopback-service/templates/loopback-service-template.xml

## Yang file  output: 

    module loopback-service {
      namespace "http://com/example/loopbackservice";
      prefix loopback-service;
    
      import ietf-inet-types {
        prefix inet;
      }
      import tailf-ncs {
        prefix ncs;
      }
    
      list loopback-service {
        key name;
    
        uses ncs:service-data;
        ncs:servicepoint "loopback-service";
    
        leaf name {
          type string;
        }
    
        // may replace this with other ways of refering to the devices.
        leaf-list device {
          type leafref {
            path "/ncs:devices/ncs:device/ncs:name";
          }
        }
    
        // replace with your own stuff here
        leaf dummy {
          type inet:ipv4-address;
        }
      }
    }

* The name variable is the service instance unique key lookup (more on that topic later too).
* The device variable is a dynamic reference in the NSO CDB to the device list.
* The dummy variable is a sample input to use in a device template, with a variable type of IP address.## XML Config Template File Output

## XML file  output: 
    <config-template xmlns="http://tail-f.com/ns/config/1.0"
                     servicepoint="loopback-service">
      <devices xmlns="http://tail-f.com/ns/ncs">
        <device>
          <!--
              Select the devices from some data structure in the service
              model. In this skeleton the devices are specified in a leaf-list.
              Select all devices in that leaf-list:
          -->
          <name>{/device}</name>
          <config>
            <!--
                Add device-specific parameters here.
                In this skeleton the service has a leaf "dummy"; use that
                to set something on the device e.g.:
                <ip-address-on-device>{/dummy}</ip-address-on-device>
            -->
          </config>
        </device>
      </devices>
    </config-template>
    (py3venv) [developer@devbox packages]$

# Developing the Service Package
We need create some XML to plug into the template. 
Create a loopback on NSO to see the XML output:

    ncs_cli -C -u admin
    config
    devices device dist-rtr01 config
    interface Loopback 100
    ip address 10.10.30.0 255.255.255.0

    show configuration
    commit dry-run outformat xml

Output: 
    data <devices xmlns="http://tail-f.com/ns/ncs">
            <device>
                <name>dist-rtr01</name>
                <config>                               
                <interface xmlns="urn:ios">            <------- |  COPY FROM HERE: <interface xmlns="urn:ios">
                    <Loopback>                                  |
                    <name>100</name>                            |
                    <ip>                                        |
                        <address>                               |
                        <primary>                               |
                            <address>10.10.30.0</address>       |
                            <mask>255.255.255.0</mask>          |
                        </primary>                              |
                        </address>                              |
                    </ip>                                       |
                    </Loopback>                                 |
                </interface>                          <-------- | TO HERE </interface>
                </config>|
            </device>
            </devices>

Edit loopback-service/templates/loopback-service-template.xml and add the XML snippet into the <config> tags (change IP address to dummy also) 

    <config-template xmlns="http://tail-f.com/ns/config/1.0"
                     servicepoint="loopback-service">
      <devices xmlns="http://tail-f.com/ns/ncs">
        <device>
          <!--
              Select the devices from some data structure in the service
              model. In this skeleton the devices are specified in a leaf-list.
              Select all devices in that leaf-list:
          -->
          <name>{/device}</name>
          <config>
                       <interface xmlns="urn:ios">
                         <Loopback>
                           <name>100</name>
                           <ip>
                             <address>
                               <primary>
                                 <address>{/dummy}</address>
                                 <mask>255.255.255.0</mask>
                               </primary>
                             </address>
                           </ip>
                         </Loopback>
                       </interface>
          </config>
        </device>
      </devices>
    </config-template>

Save the XML template text file and exit the NSO configuration session, discarding the pending NSO device change.In your bash terminal navigate to the loopback-service/src/ directory, and issue the make command to compile the YANG data model. After updating the XML and compiling the YANG model, load in the new service package

    cd loopback-service/src/
    make
    ncs_cli -C -u admin
    packages reload

# Creating a Service Instance
With the service package loaded in NSO, you can start creating service instances.
Enter config mode.Use the new loopback-service command to create a service instance with a name called test and populate the variables device with dist-rtr01 and dummy with 192.168.1.1

    config
    loopback-service test
    device dist-rtr01
    dummy 192.168.1.1
    top
    commit dry-run outformat native
    show configuration
    commit

# Redeploying a Service
An out-of-band change might be done on your network that NSO is unaware of, and conflicts with your service. View NSO's understanding of the device. Check the CDP snapshot of the device to understand if NSO is not aware of the change yet.

    show running-config devices device dist-rtr01 config interface Loopback

Sync config from device so that NSO learns about the change and redeploy your config

    devices sync-from dry-run
    devices sync-from
    loopback-service test re-deploy dry-run
    loopback-service test re-deploy
