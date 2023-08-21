This is a deeper look at how services are implemented. It can be skipped if you are not interested in how the sausage is made.

The main ways to develop a service are:
* Pure templates
* Java (and templates)
* Python (and templates)

Sample files:
service.yang: This file describes the interface to the service and includes all the fields and their types. It can also contain constraints such as pattern or must expressions

    module simple-service {
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

simple-service-template.xml: This template shows the mapping between the service model described in the YANG file and the desired device configuration. The {./secret} blocks are simple XPath statements referencing the service model's corresponding fields.

    <config-template xmlns="http://tail-f.com/ns/config/1.0" servicepoint="simple-service">
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
