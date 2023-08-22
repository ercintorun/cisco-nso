NSO service is a method to abstract and automate a task you want to de repeteadly. 

## Creating the Service package
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

## XML Config Template File Output

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
