https://developer.cisco.com/learning/tracks/get_started_with_nso/nso-labs/yang-nso-101/using-data-models-in-nso/

# Custom Yang Model

Download yang model from: 

https://github.com/CiscoDevNet/nso-lab-files
 
NSO loads the packages from the packages subdirectory of the running directory, so you need to copy the package to the nso-run/packages folder:
    
    (py3venv) [developer@devbox ~]$ cp -a lab-files/yang-nso-101/custom-yang-package nso-run/packages/

Before NSO can use a new YANG file, the file must be compiled into a binary form that NSO understands. An easy way for you to achieve this is with the make command. Run the following command that compiles the newly added package:

    (py3venv) [developer@devbox ~]$ make -B -C nso-run/packages/custom-yang-package/src/
    make: Entering directory `/home/developer/nso-run/packages/custom-yang-package/src'
    mkdir -p ../load-dir
    /home/developer/nso/bin/ncsc  `ls custom-yang-package-ann.yang  > /dev/null 2>&1 && echo "-a custom-yang-package-ann.yang"` \
                  -c -o ../load-dir/custom-yang-package.fxs yang/custom-yang-package.yang
    make: Leaving directory `/home/developer/nso-run/packages/custom-yang-package/src'

Person yang model: 
https://github.com/CiscoDevNet/nso-lab-files/blob/main/yang-nso-101/custom-yang-package/src/yang/custom-yang-package.yang 

    list person {
      key email;
    
      leaf first-name {
        tailf:info "Person's first name";
        mandatory true;
        type string;
      }
    
    
      leaf last-name {
        tailf:info "Person's last name";
        mandatory true;
        type string;
      }
    
    
      leaf date-of-birth {
        tailf:info "Date of birth as dd/mm/yyyy";
        type string {
          pattern "[0-9][0-9]/[0-9][0-9]/[0-9][0-9][0-9][0-9]";
        }
      }
    
    
      leaf email {
        tailf:info "Contact email";
        mandatory true;
        type string;
      }
    
    
      leaf job-role {
        tailf:info "Persons job role within organization";
        type enumeration {
          enum admin;
          enum developer;
          enum manager;
        }
      }
    }

## Add custom data to CDB

    developer@ncs(config)#  person john@cisco.com first-name John last-name Smith date-of-birth 01/01/1980 job-role admin
    developer@ncs(config-person-john@cisco.com)#commit dry-run 
    developer@ncs(config-person-john@cisco.com)#commit 

Configuration data is now persisted in the CDB and is available to be used for automation later.
Custom YANG models are usually included as service packages in NSO. In this example, the person data could be used for the login or MOTD banner service on your network devices to show contact information.
