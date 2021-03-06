# Tutorial of MongoDB and Iperf using Docker

This tutorial will guide you through the deployment of a Mongo DB [Docker](https://www.docker.com) container.

## Common Requirements

You will need three Open Baton Components:

* NFVO
* Docker VNFM
* Docker VIM Driver

and a reachable Docker engine, usually where the Open Baton platform is running, but not necessarily.


## Setup

In order to start manually the Docker VNFM and VIM Driver, please follow these README: [VNFM][docker-vnfm] and [VIM Driver][docker-driver].
When the VNFM and the VIM Driver are running, the NFVO Dashboard should show in the VNF Manager page the Docker VNFM and in the installed vim driver the Docker Vim Driver.

After this you should be able to register the Docker Engine as a PoP:

```json
{
  "name": "vim-instance",
  "authUrl": "unix:///var/run/docker.sock",
  "tenant": "1.32",
  "username": "admin",
  "password": "openbaton",
  "type": "docker",
  "location": {
    "name": "Berlin",
    "latitude": "52.525876",
    "longitude": "13.314400"
  }
}
```

* **authUrl** either you pass the unix socket, in this case will use the socket running locally to the vim driver or the host connection string for remote execution
* **tenant** in the tenant you can specify the api version used by the chosen docker engine
* **type** is docker if the Docker VIM Driver uses the default type value

after registering your PoP, you should be able to see all images and networks in the PoP page of the NFVO Dashboard

After this step you can pass to the VNF Package. If you don't know what it is or how to create one follow [this guide](vnf-package). However, the content of the _Metadata.yaml_ and of the _vnfd.json_ can be as follow:

## MongoDB  VNF Package

```json
{
    "name": "MongoDB",
    "vendor": "TUB",
    "version": "0.2",
    "lifecycle_event": [],
    "configurations": {
      "configurationParameters": [{
        "confKey":"KEY",
        "value":"Value"
      }],
      "name": "mongo-configuration"
    },
    "virtual_link": [{
      "name": "new-network"
    }],
    "vdu": [{
      "vm_image": [
      ],
      "scale_in_out": 2,
      "vnfc": [{
        "connection_point": [{
          "virtual_link_reference": "new-network"
        }]
      }]
    }],
    "deployment_flavour": [{
      "flavour_key": "m1.small"
    }],
    "type": "mongodb",
    "endpoint": "docker"
  }
```

Where:

* The _**Virtual Link**_ will be a new Docker Network created if not existing.
* The _**flavour_key**_ must be set to _m1.small_ (at the moment)  
* The _**vm_image**_ will be filled by the _metadata_ image name (see next section)  

##### The Metadata.yaml

```yaml
name: MongoDB
description: MongoDB
provider: TUB
nfvo_version: 6.0.0
vim_types:
 - docker
```

Where:

* **vim_types** must have docker (pointing to the Docker VIM Driver)

You can then create and upload the tar file to the VNF Package page of the Dashboard and you should have a VNFD called in this case MongoDB that represent a MongoDB Container.

Afterwards, from the NSD page, you can create a NSD using the form and selecting the MongoDB VNF.

## Iperf NSD

This tutorial will guide you through the deployment of Iperf client and server in two different [Docker](https://www.docker.com) containers.

This tutorial is different since you don't have to create a VNFPackage but you need to build two Docker images.

Create a file called Dockerfile for the Iperf client image that looks like:

```docker
FROM networkstatic/iperf3

ENTRYPOINT  iperf3 -c $SERVER_HOSTNAME -t 300
```
and then you can build it as:

```bash
docker build -t iperfclient .
```

Do the same for the Iperf server:

```docker
FROM networkstatic/iperf3

EXPOSE 5201

ENTRYPOINT ["iperf3", "-s"]
```

and build it with:

```bash
docker build -t iperfserver .
```

After this, you need to refresh the VIM on order to see the new images: go to Manage PoPs&rarr;PoP instances&rarr;ID and click on the two round arrows close to the name.
This will trigger the Vim refresh.  
Afterwards, you need to create a NSD. From the NFVO dashboard, go to Catalogue&rarr;NS Descriptors&rarr;On Board NSD&rarr;Upload JSON and paste this JSON content:

```json
{
  "name": "iperf",
  "vendor": "FOKUS",
  "version": "docker",
  "vld": [{
    "name": "new-network"
  }],
  "vnfd": [
    {
      "name":"iperf-server",
      "vendor":"FOKUS",
      "version":"docker",
      "deployment_flavour":[
        {
          "flavour_key":"m1.small"
        }
      ],
      "vdu":[
        {
          "vm_image":["iperfserver:latest"],
          "scale_in_out":3,
          "vnfc":[
        {
          "connection_point":[
            {
              "virtual_link_reference":"new-network"
            }
          ]
        }
          ]
        }
      ],
      "virtual_link":[
        {
           "name":"new-network"
        }
      ],
      "type":"server",
      "endpoint":"docker"
    },
    {
      "name":"iperf-client",
      "vendor":"FOKUS",
      "version":"docker",
      "vdu":[
        {
          "vm_image":["iperfclient:latest"],
          "scale_in_out":3,
          "vnfc":[
        {
          "connection_point":[
            {
              "virtual_link_reference":"new-network"
            }
          ]
        }
          ]
        }
      ],
      "deployment_flavour":[
        {
          "flavour_key":"m1.small"
        }
      ],
      "virtual_link":[
        {
           "name":"new-network"
        }
      ],
      "requires": {
        "iperf-server": {
        "parameters":["hostname"]
        }
      },
      "type":"client",
      "endpoint":"docker"
    }
  ]
}
```

and launch the NSD.

[docker-vnfm]: vnfm-docker
[docker-driver]: docker-driver
