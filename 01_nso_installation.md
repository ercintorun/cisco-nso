# NSO INSTALLATION MODES

System Install Use when installing NSO for a centralized, "always-on", production grade purpose. You configure it as a system daemon that starts and ends with the underlying operating system. The default credentials and the application installation locations align with a typical Linux application function.

Local Install Use for development and evaluation purposes. The installation unpacks all the application components, including documentation and examples, and you instantiate and start instances of NSO on demand. You use a Local Install on a single workstation to have multiple, unrelated instances of NSO for different demos (not at the same time). This install includes default credentials for ease of use and you typically install it in an easily accessible folder.

# INSTALL NCS on REDHAT
## Installing Prerequisites 
without development tools netsim gives "make" errors. 
python3-setuptools by default installed on redhat

    yum groupinstall 'Development Tools' 
    yum install java-1.8.0-openjdk 
    yum install ant
    yum install python3-setuptools

    wget https://bootstrap.pypa.io/get-pip.py
    python3 ./get-pip.py

    pip3 install paramiko
    pip3 install requests

## Download the packages
    [root@cisco-nso lab]# ls
    ncs-6.1-cisco-ios-6.92.8-freetrial.signed.bin  nso-6.1-freetrial.linux.x86_64.signed.bin
    ncs-6.1-cisco-iosxr-7.49-freetrial.signed.bin

## Install NSO
    sh nso-6.1-freetrial.linux.x86_64.signed.bin
    mkdir nso-6.1 
    sh nso-6.1.linux.x86_64.installer.bin --local-install nso-6.1

## Extract NEDS
    sh ncs-6.1-cisco-ios-6.92.8-freetrial.signed.bin
    sh ncs-6.1-cisco-iosxr-7.49-freetrial.signed.bin

    cd /home/lab/nso-6.1/packages/neds 
    tar -zxvf /home/lab/ncs-6.1-cisco-ios-6.92.8.tar.gz
    tar -zxvf /home/lab/ncs-6.1-cisco-iosxr-7.49.tar.gz 

You can select only one version of ned for a device type 
    source /home/lab/nso-6.1/ncsrc 

## Create an instance of NSO 
    cd /home/lab/nso-6.1/
    ncs-setup --package /home/lab/nso-6.1/packages/neds/cisco-ios-cli-6.92/ \
    --package /home/lab/nso-6.1/packages/neds/cisco-iosxr-cli-7.49/ \
    --package /home/lab/nso-6.1/packages/neds/cisco-ios-cli-6.92/ \
    --package /home/lab/nso-6.1/packages/neds/juniper-junos-nc-3.0/ \
    --package /home/lab/nso-6.1/packages/neds/cisco-nx-cli-3.0/ \
    --dest nso-instance1

## Start nso instance
    cd /home/lab/nso-6.1/nso-instance1
    ncs
    ncs --status | grep status 
    tail -f logs/devel.log
    ncs --stop 

## Playing with example Labs
    source /home/lab/nso-6.1/ncsrc
    cd /home/lab/nso-6.1/examples.ncs/getting-started/using-ncs/1-simulated-cisco-ios
    ncs-netsim create-network /home/lab/nso-6.1/packages/neds/cisco-ios-cli-3.8 3 ios

    [root@cisco-nso 1-simulated-cisco-ios]# ncs-netsim create-network /home/lab/nso-6.1/packages/neds/cisco-ios-cli-3.8 3 ios
    DEVICE ios0 CREATED
    DEVICE ios1 CREATED
    DEVICE ios2 CREATED

    ncs-setup --dest .
    nst-netsim start
    ncs
