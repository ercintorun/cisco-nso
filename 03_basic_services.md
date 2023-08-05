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
