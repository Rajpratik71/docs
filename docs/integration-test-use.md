# How to use the Integration Tests

## Overview

This project provides integration tests for OpenBaton. 
Eleven tests are run.

## Test descriptions

1. scenario-dummy-iperf
2. scenario-many-dependencies
3. scenario-real-iperf
4. scenario-complex-ncat
5. scenario-scaling
6. error-in-configure
7. error-in-instantiate
8. error-in-start
9. error-in-terminate
10. wrong-lifecycle-event
11. user-project-test
12. stress-test

**scenario-dummy-iperf** uses the [Dummy VNFM][vnfm-dummy] to simulate a VNFM and therefore tests the communication between NFVO and VNFM. 
It does not actually deploy a network service. The fake network service is a simple iperf scenario with one server and one client. 

**scenario-many-dependencies** also uses the Dummy VNFM but its fake network service is a little bit more complex in the sense that it has many VNFD with many dependencies between them. 

The test **scenario-real-iperf** actually deploys a network service on openstack. 
It consists of two VNFD and deploys one iperf server and two iperf clients. The clients contact the server. 

The test **scenario-complex-ncat** deploys a more complex network service on openstack. 
Five virtual machines will be running and acting as peers. 
The following picture shows the architecture in which the peers connect to each other using ncat and send their ip address. 
A peer at the beginning of an arrow acts as an ncat client and connects to the peer at the end of the arrow. 
The receiving peer stores the ip address of the sender so that it is possible to verify which peer connected to which. 
The two colours represent the two different networks on which the peers are running and connecting. 
Blue is the network *private* and black *private2*.

![Complex scenario][complex-ncat]

The test **scenario-scaling** tests the scaling function of Openbaton. 
It starts by deploying two VMs one acting as an ncat server and one as an ncat client which sends his ip address to the server so that it is possible to check if the client actually connected to the server. 
Then it executes some scaling functions like scale out and scale in and checks if new instances of the server and the client are deployed. Cases like scale in on just one instance and scale out on the maximum number of instances are included. 
It also examines if the client instances are provided with the ip addresses of the new server instances, so that they are able to connect to them. 
To see detailed information about which scaling functions are executed exactly please refer to the *scenario-scaling.ini* file in the project.

The tests **error-in-configure**, **error-in-instantiate**, **error-in-start**, **error-in-terminate** each deploy a network service from a NSD which contains a failing script in the particular lifecycle event and tests if the NFVO handles it correctly. 

The test **wrong-lifecycle-event** tries to onboard a NSD to the NFVO which contains an undefined lifecycle event. The test will pass if the onboarding is not successful. 

The **user-project-test** checks if the NFVO handles user and project management correctly. It adds and deletes users, projects and a vim instance from different 
user perspectives. This test can be executed without a VNFManager. 

The **stress-test** checks if the NFVO can handle a large number of NSR deployments from the same or different NSDs at the same time. This test uses the Dummy-VNFM. For this test you should set the property *nfvo.vmanager.executor.maxpoolsize* to a large number (e.g. 200) in the /etc/openbaton/openbaton.properties file.

In most of the tests a vim instance and a network service descriptor are stored on the orchestrator and the network service launched. 
If that is successful, the network service is stopped and the network service record, network service descriptor and the vim instance are removed. 
In the cases of the **scenario-real-iperf**, **scenario-complex-ncat** and **scenario-scaling** test also the service itself is tested, i.e. if iperf is running and the clients can connect to the server. Therefore the integration tests will execute some scripts for testing on the virtual machines. 

## Requirements

1. A running [NFVO][nfvo-github] with the OpenStack vim driver and Test vim driver
2. A running [Generic VNFM][generic-vnfm-github]
3. A running [Dummy VNFM][dummy-vnfm-amqp-github]
4. OpenStack

## Installation and configuration

Use git to clone the integration-test project to your machine. 
In *integration-tests/src/main/resources* is a file named integration-tests.properties. 
Open it and set the property values according to your needs. 

| Field          				| Value       																|
| -------------   				| -------------:																|
| nfvo-ip  					| The ip of the machine on which the NFVO you want to use is running |
| nfvo-port					| The port on which the NFVO is running |
| nfvo-usr					| The username required for logging in the NFVO |
| nfvo-pwd                                      | The password required for logging in the NFVO |
| nfvo-project-id                               | The id of the project that the integration tests shall use |
| nfvo-ssl-enabled				| Set this to *true* if the NFVO uses SSL |
| local-ip					| The ip of the machine on which the integration test is running |
| clear-after-test                              | If set to *true*, the NFVO will be cleared of all the remaining NSRs, NSD, VNFPackages and Vim-Instances left from previous test |
| integration-test-scenarios                    | Here you can specify a folder in which you can put integration test scenarios. The *.ini* files in this folder overwrite the ones in the projects resource folder |
| external-properties-file   | If you want to use another file for fetching the properties. It is already preset to */etc/openbaton/integration-tests/integration-tests.properties*. If it does not exist it will not be used. |


After that you also need a keypair for OpenStack. 
Import a key pair in the OpenStack dashboard, give it a name and assign the public key of the host, on which the integration tests will run, to it. 
The next step is to create a vim file. 
Here is an example where you just have to change some fields. 
```json
{
  "name":"vim-instance",
  "authUrl":"http://your-openstack-url",
  "tenant":"the tenant you use",
  "username":"openstack username",
  "password":"openstack password",
  "keyPair":"the name of the imported key pair",
  "securityGroups": [
    "default"
  ],
  "type":"openstack",
  "location":{
    "name":"your location",
    "latitude":"the latitude",
    "longitude":"the longitude"
  }
}
```

Name the vim file *real-vim.json* and add it to the folder *integration-tests/src/main/resources/etc/json_file/vim_instances/* in the project.
In the folder *integration-tests/src/main/resources/etc/json_file/network_service_descriptors* of the project you will find a file named NetworkServiceDescriptor-iperf-real.json and one named NetworkServiceDescriptor-complex-iperf.json. 
Download the [ubuntu-14.04-server-cloudimg-amd64-disk1][ubuntu-image] image and store it in OpenStack. 

The scenario *error-in-terminate.ini* needs some special configuration in the NFVO if you want to run it. Change **nfvo.delete.vnfr.wait** to **true** in */etc/openbaton/openbaton.properties* before starting the NFVO.

Then use a shell to navigate into the project's root directory. 
Execute the command *./integration-tests.sh compile*.

## Start the integration test

Before starting the integration tests be sure that the NFVO, Generic VNFM and Dummy VNFM you want to use are already running. 
Then start the test by executing *./integration-tests.sh start*.
It is possible to specify the test scenarios you want to run so that not every test in the */src/main/resources/integration-test-scenarios* folder is executed. 
Therefore use additional command line arguments while starting the integration tests. Every scenario occuring as an argument will be executed. For example *./integration-tests.sh start scenario-real-iperf.ini scenario-scaling.ini* will just execute the tests described in the files *scenario-real-iperf.ini* and *scenario-scaling.ini* located in the folder */src/main/resources/integration-test-scenarios*.
If you do not pass any command line arguments, every available scenario will be executed. To see which scenarios are available execute *./integration-tests.sh list*.

## Test results

While the tests are running they will produce output to the console. This output will be logged in the file integration-test.log which is in the project's root directory. 
If a test finished it will either tell you that it passed successfully or not. 
If it did not pass correctly you will find the reason in the log output. 


## Write your own integration tests
Please refer to [this page][integration-test-write].

<!---
References

-->

[complex-ncat]:images/complex-ncat.png
[nfvo-installation]:nfvo-installation
[vnfm-generic]:vnfm-generic
[nfvo-github]:https://github.com/openbaton/NFVO
[generic-vnfm-github]:https://github.com/openbaton/generic-vnfm
[dummy-vnfm-amqp-github]:https://github.com/openbaton/dummy-vnfm-amqp
[vnf-descriptor]:vnf-descriptor
[integration-test-write]:integration-test-write
[test-plugin]:https://github.com/openbaton/test-plugin
[openstack-plugin]:https://github.com/openbaton/openstack-plugin
[ubuntu-image]:https://uec-images.ubuntu.com/releases/14.04/release/ubuntu-14.04-server-cloudimg-amd64-disk1.img

<!---
Script for open external links in a new tab
-->
<script type="text/javascript" charset="utf-8">
      // Creating custom :external selector
      $.expr[':'].external = function(obj){
          return !obj.href.match(/^mailto\:/)
                  && (obj.hostname != location.hostname);
      };
      $(function(){
        $('a:external').addClass('external');
        $(".external").attr('target','_blank');
      })
</script>
