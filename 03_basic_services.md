## Create Service Package
Services simplifies device configuration by automating service provisioning.
To add a service, you need to create a service package, which will have some pre-built files and directories. 

    ncs-make-package --service-skeleton template --dest ~/src/packages/simple-service simple-service
    printf 'module simple-service {
      namespace "http://com/example/simpleservice";
      prefix simple-service;
      import tailf-ncs { prefix ncs; }
      list simple-service {
        key name;
        uses ncs:service-data;
        ncs:servicepoint simple-service;
        leaf name {
          type string;
        }
        leaf device {
          type leafref {
            path "/ncs:devices/ncs:device/ncs:name";
          }
        }
        leaf secret {
          type string;
        }
      }
    }
    ' > ~/src/packages/simple-service/src/yang/simple-service.yang
    printf '<config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="simple-service">
      <devices xmlns="http://tail-f.com/ns/ncs">
        <device>
          <name>{./device}</name>
          <config>
            <enable xmlns="urn:ios">
              <password>
                <secret>{./secret}</secret>
              </password>
            </enable>
          </config>
       </device>
      </devices>
    </config-template>
    ' > ~/src/packages/simple-service/templates/simple-service-template.xml
    make -C ~/src/packages/simple-service/src
    echo "packages reload" | ncs_cli -C -u admin

We just added a service package to NSO named simple-service available which sets the enable password on a Cisco IOS device. Check the package installation status:

    show packages package oper-status

## Configuring a Service 
Creates a service instance named test1 that configures the device ios0 to have the secret mypasswd.

    config 
    simple-service test1 device ios0 secret mypasswd
    show configuration
    commit dry-run
    commit dry-run outformat native
    commit
    
## Modifying a Service 
While in service context, change the secret to securepasswd: 

    secret securepasswd
    commit dry-run
    commit

Leave service context and get modifications:
    top
    simple-service test1 get-modifications

The next one is re-deploy, which tries to deploy the service again. This is especially useful if manual changes have been made to the device configuration and you want to reassert the service configuration:
    simple-service test1 re-deploy

Delete service: 
    no simple-service test1
    commit dry-run
    commit
