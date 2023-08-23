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

### Define a YANG module

Sample yang module: (You can find the explanations of the data-types etc. below the yang module) 

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
    
      typedef port-type {
        type union {
          type uint16 {
            range "0..65535";
          }
          type enumeration {
            enum "ssh" {value 22;}
            enum "telnet" {value 23;}
            enum "https" {value 443;}
            enum "http" {value 80;}
            enum "ftp" {value 21;}
            enum "pop3" {value 110;}
            enum "smtp" {value 25;}
            enum "tftp" {value 69;}
            enum "domain" {value 53;}
            enum "ntp" {value 123;}
          }
        }
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
              description "Sequence number of an ACE";
              leaf action {
                  type enumeration {
                      enum permit;
                      enum deny;
                 }
                  description "Action to take with the packet";
              }
              leaf source-ip {
                  type inet:ipv4-address; 
                  description "Source IPv4 address.";
              }
              leaf destination-ip {
                  type inet:ipv4-address;
                  description "Destination IPv4 address.";
              }
              leaf destination-port {
                  type port-type;
              }
          }
      }
    }

On the example think of "list" as a dictionary in python, it has key and the value part. 
When you want to limit a number of options, you can use the enumeration statement, like "permit/deny" in the example above

The NSO installation folder contains many useful YANG modules, which consist of custom predefined YANG data types. For example, the ietf-inet-types holds a group of commonly used predefined YANG data types for addresses and other internet-related objects and features. It is imported in the example above right after "prefix" with "import ietf-inet-types......" and "ietf:ipv4-address" type is used for ip address entry in the acl. 

Sometimes, built-in YANG data types do not suit your needs, and you are unable to find modules that you can import with suitable data types. In this case, you can build your own.The typedef statement describes a data type that is derived from a base types and supports a hierarchy of derived types. The base type can be a derived type itself or a YANG built-in type. You can use the derived type as an argument for the type statement. The typedef statements are positioned just after the revision statement.

A useful tool that you can use to check that the validity of a YANG syntax is pyang.If there are errors in the file, they are displayed in the output, receiving no output means that all is well. 

    pyang --strict ip-access-list.yang
    
Your module is starting to take shape but you will must compile it before NSO can use it 

    (py3venv) [developer@devbox yang]$ make -C ..
    /home/developer/nso/bin/ncsc  `ls ip-access-list-ann.yang  > /dev/null 2>&1 && echo "-a ip-access-list-ann.yang"` \
                  -c -o ../load-dir/ip-access-list.fxs yang/ip-access-list.yang

The output should not contain any errors. If there are any, update the YANG file before continuing.

### Identities 
To define a new globally unique, untyped, and abstract identity, use the identity statement. The identity, similar to typedef, can also be based on an already existing base identity or created from scratch. The name of the identity is the identifier for the identity's argument.

The identity statement is similar to the enumeration statement and enables you to limit the input options to predefined identities. The advantage of the identity is that it enables you to create new (identity) types, without changing the base identity. To create or modify other YANG types, you must change the referenced (original) model (type) first. To reference the identity inside a data model, use the identityref built-in datatype.

The basic ACL module that you created for this Lab is nearly complete. However, you must still add an option for the protocol. Adding this option enables you to filter traffic based on the protocol (TCP, UDP, Internet Control Message Protocol ICMP, and so on). You can then make the ACL rules (entries) as specific as you need them.

First, create a new .yang extension file for the base identity of the protocol.

    (py3venv) [developer@devbox yang]$ touch protocol-base.yang
    
    module protocol-base {
      namespace "http://example.com/ns/yang/protocol-base";
      prefix protocol;
      description "The base module for the protocol";
    
      identity protocol {
        description "Custom identity for the protocol";
      }
    
    
      identity tcp {
        base protocol;
        description "The Transmission Control Protocol";
      }
    }

Save the file and compile it

    (py3venv) [developer@devbox yang]$ make -C ..
    /home/developer/nso/bin/ncsc  `ls ip-access-list-ann.yang  > /dev/null 2>&1 && echo "-a ip-access-list-ann.yang"` \
                  -c -o ../load-dir/ip-access-list.fxs yang/ip-access-list.yang
    /home/developer/nso/bin/ncsc  `ls protocol-base-ann.yang  > /dev/null 2>&1 && echo "-a protocol-base-ann.yang"` \
                  -c -o ../load-dir/protocol-base.fxs yang/protocol-base.yang
              
Now you can import the protocol-base module into the ip-access-list module under the already imported ietf-inet-types module.

    import ietf-inet-types {
    ...
    import protocol-base {
     prefix  protocol;
    }
    ...
    organization
   
In ip-access-list.yang, create the new identities beneath the typedef port type and just above the container node.

    typedef port type {
    ...
    identity udp {
      base protocol:protocol;
      description "The User Datagram Protocol";
    }
    identity icmp {
     base protocol:protocol;
     description "The Internet Control Message Protocol";
    }
    \\ ..
    container acl {
   
Now, create a new leaf underneath the destination-port, for the protocol with the identityref type which points to the base identity.

    leaf destination-port {
    ...
    leaf protocol {
      type identityref {
       base protocol:protocol;
      }
      description "Protocol.";
    }  
  
Save the file and compile it with the YANGPATH set as follows:

    (py3venv) [developer@devbox yang]$ YANGPATH="./yang" make -C ..
    /home/developer/nso/bin/ncsc  `ls ip-access-list-ann.yang  > /dev/null 2>&1 && echo "-a ip-access-list-ann.yang"` \
                  -c -o ../load-dir/ip-access-list.fxs yang/ip-access-list.yang
    /home/developer/nso/bin/ncsc  `ls protocol-base-ann.yang  > /dev/null 2>&1 && echo "-a protocol-base-ann.yang"` \
                  -c -o ../load-dir/protocol-base.fxs yang/protocol-base.yang

The YANGPATH="./yang" part instructs the build system to also search the YANG folder for any required modules. Without it, you may get an error that the system cannot find the module protocol-base.

Reload the packages into the NSO and check the result in the CLI.

    admin@ncs(config)# acl entry 10 protocol ?
    Possible completions:
      icmp tcp  udp
  
When statement 

Since the ICMP protocol operates on the network layer, it shouldn't support the destination-port option. You can restrict this with the when statement.
Add a when statement inside the destination-port leaf. The xpath must go outside the current (destination-port) node and then make sure that the chosen protocol is not icmp.

    leaf destination-ip {
    ...
    leaf destination-port {
        when "../protocol != 'acl:icmp'";
        type port-type;
    }
    leaf protocol {
    ...

Compile the file with the YANGPATH="./yang" set and then once again reload the packages.

    (py3venv) [developer@devbox yang]$ YANGPATH="./yang" make -C ..
    /home/developer/nso/bin/ncsc  `ls ip-access-list-ann.yang  > /dev/null 2>&1 && echo "-a ip-access-list-ann.yang"` \
                  -c -o ../load-dir/ip-access-list.fxs yang/ip-access-list.yang


The comple access-list yang file is now: 
    module ip-access-list {
      namespace "http://example.com/ns/yang/ip-access-list";
      prefix acl;
    
      import ietf-inet-types {
        prefix inet;
      }
    
    
      import protocol-base {
        prefix protocol;
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
    
    
      typedef port-type {
        type union {
          type uint16 {
            range "0..65535";
          }
          type enumeration {
            enum "ssh" {value 22;}
            enum "telnet" {value 23;}
            enum "https" {value 443;}
            enum "http" {value 80;}
            enum "ftp" {value 21;}
            enum "pop3" {value 110;}
            enum "smtp" {value 25;}
            enum "tftp" {value 69;}
            enum "domain" {value 53;}
            enum "ntp" {value 123;}
          }
        }
      }
      identity udp {
         base protocol:protocol;
         description "The User Datagram Protocol";
      }
      identity icmp {
        base protocol:protocol;
        description "The Internet Control Message Protocol";
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
            description "Name of maintainers
       working on the ACL";
          }
          list entry {
              key "number";
              leaf number {
                  type uint16;
              }
              description "Sequence number of an ACE";
              leaf action {
                  type enumeration {
                      enum permit;
                      enum deny;
                 }
                  description "Action to take with the packet";
              }
              leaf source-ip {
                  type inet:ipv4-address;
                  description "Source IPv4 address.";
              }
              leaf destination-ip {
                  type inet:ipv4-address;
                  description "Destination IPv4 address.";
              }
              leaf destination-port {
                  when "../protocol != 'acl:icmp'";
                  type port-type;
              }
              leaf protocol {
                  type identityref {
                     base protocol:protocol;
                  }
                  description "Protocol.";
              }
          }
      }
    } 

### Result

    admin@ncs(config)# acl acl-description "Devnet"
    admin@ncs(config)# acl maintainers John
    admin@ncs(config)# acl entry 30 action permit source-ip 10.20.30.1 destination-ip 10.10.10.1 protocol tcp destination-port ssh
    admin@ncs(config-entry-30)# commit dry-run
    cli {
        local-node {
            data  acl {
                 -    acl-description Test1;
                 +    acl-description Devnet;
                 -    maintainers
             [ Alice Bob ];
                 +    maintainers
             [ Alice Bob John ];
                 +    entry 30 {
                 +        action permit;
                 +        source-ip 10.20.30.1;
                 +        destination-ip 10.10.10.1;
                 +        protocol tcp;
                 +        destination-port ssh;
                 +    }
                  }
        }
    }
