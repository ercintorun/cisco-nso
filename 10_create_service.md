NSO service is a method to abstract and automate a task you want to de repeteadly. 

## Creating the Service package
Service packages are a collection of structured files and folders that NSO loads in to the application to extend NSO with new functionality

    cd nso-instance/packages/
    ls
    ncs-make-package -h
    ncs-make-package --service-skeleton template loopback-service
