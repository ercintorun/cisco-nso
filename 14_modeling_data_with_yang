# Modeling data with YANG
https://developer.cisco.com/learning/tracks/get_started_with_nso/nso-labs/yang-nso-201/creating-a-new-model/ 

## Create a Service Package: 

You will  need a package that holds your YANG model (or models). 
The simplest way to do so is by creating a service package with the ncs-make-package command,

    cd packages && ncs-make-package --service-skeleton template my-models

NSO can't use YANG files in their original form, they have to be compiled first. The ncs-make-package command created my-models/src/_directory that holds all the files that need compilation, and a Makefile. The Makefile contains the recipes to easily build the source files. It instructs the make command to look for all the YANG files under the src/yang/ path and compile them into an NSO-compatible (binary) format.

The ncs-make-package command also creates some additional files that you will not be needing. Using the rm command, remove the following files:

    rm -f my-models/templates/* my-models/src/yang/*
    cd my-models/src/yang 

## Create a New Model

### Define a YANG module

Sample yang module: 

    module ip-access-list {
      namespace "http://example.com/ns/yang/ip-access-list";
      prefix acl;
    
      import ietf-inet-types {
        prefix inet;
      }
    
    
      organization
        "Example, Inc.";
    
    
      contact
        "Example, Inc.
         Customer Service
         E-mail: cs-yang@example.org";
    
    
      description
        "Access Control List (ACL) YANG model.";
    
    
      revision 2021-07-06 {
        description
          "Initial revision";
      }
      // ...
      container acl {
        description
          "Access Control Lists";
          leaf acl-description{
            type string{
            length  "0..64";
            pattern "[0-9a-zA-Z]*";
            }
            description "Purpose of ACL";
          }
          leaf-list maintainers {
            type string;
            description "Name of maintainers working on the ACL";
          }
          list entry {
              key "number";
              leaf number {
                  type uint16;
              }
              description "Sequence number of an ACL";
          }
      }
    }

On the example think of "list" as a dictionary in python, it has key and the value part. 

A useful tool that you can use to check that the validity of a YANG syntax is pyang.If there are errors in the file, they are displayed in the output, receiving no output means that all is well. 

    pyang --strict ip-access-list.yang
    
Your module is starting to take shape but you will must compile it before NSO can use it 

    (py3venv) [developer@devbox yang]$ make -C ..
    /home/developer/nso/bin/ncsc  `ls ip-access-list-ann.yang  > /dev/null 2>&1 && echo "-a ip-access-list-ann.yang"` \
                  -c -o ../load-dir/ip-access-list.fxs yang/ip-access-list.yang

The output should not contain any errors. If there are any, update the YANG file before continuing.
     
