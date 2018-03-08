# KumuluzEE Node.js Discovery — register service
The objective of this sample is to show how to register service with Consul and etcd using KumuluzEE Node.js Discovery library.


## Requirements

Node version >= 8.0.0
```
$ node --version
```
Note: if you are running service on Debian operating system, you will need to install `nodejs-legacy`:

```
$ sudo apt-get install nodejs-legacy
```


## Prerequisites



**Running Consul**

First you should check if you have installed Consul, by typing following command:
```
$ consul version
```

Note that such setup with Consul running in development mode is not viable for production environments, but only for developing purposes. Here is an example on how to quickly run a local Consul agent in development mode:
```
 $ consul agent -dev -ui
```

**Running etcd**

Note that such setup with only one etcd node is not viable for production environments, but only for developing purposes. Here is an example on how to quickly run an etcd instance with docker:
```
 $ docker run -d -p 2379:2379 \
     --name etcd \
     --volume=/tmp/etcd-data:/etcd-data \
     quay.io/coreos/etcd:latest \
     /usr/local/bin/etcd \
     --name my-etcd-1 \
     --data-dir /etcd-data \
     --listen-client-urls http://0.0.0.0:2379 \
     --advertise-client-urls http://0.0.0.0:2379 \
     --listen-peer-urls http://0.0.0.0:2380 \
     --initial-advertise-peer-urls http://0.0.0.0:2380 \
     --initial-cluster my-etcd-1=http://0.0.0.0:2380 \
     --initial-cluster-token my-etcd-token \
     --initial-cluster-state new \
     --auto-compaction-retention 1 \
     -cors="*"
```

## Tutorial

This tutorial will guide you through the steps required to register KumuluzEE microservice with Consul and etcd. We will use a simple [Customer REST](https://github.com/kumuluz/kumuluzee-samples/tree/master/jax-rs) service with the following resources:

-   GET  [http://localhost:8080/v1/customers](http://localhost:8080/v1/customers)  \- list of all customers
-   GET  [http://localhost:8080/v1/customers/{customerId}](http://localhost:8080/v1/customers/%7BcustomerId%7D)  – details of customer with ID {customerId}
-   POST  [http://localhost:8080/v1/customers](http://localhost:8080/v1/customers)  – add a customer
-   DELETE  [http://localhost:8080/v1/customers/{customerId}](http://localhost:8080/v1/customers/%7BcustomerId%7D)  – delete customer with ID {customerId}

First install dependencies using npm:

```
$ npm install
```

### Register service
Import `KumuluzeeDiscovery`, discovery source client and initialize 

```javascript
import KumuluzeeDiscovery from 'kumuluzee-nodejs-discovery';

// ...

const register = async () => {
  await KumuluzeeDiscovery.initialize({ extension: 'etcd' }); // or 'consul'

  KumuluzeeDiscovery.registerService();
}

register()

// ...
```

### Add required configuration for the service discovery

You can add configuration using any KumuluzEE configuration source.

For example, you can use config.yml file, placed in config folder:

```yml
kumuluzee:
  name: customer-service
  env:
    name: dev
  version: 1.0.0
  server:
    base-url: http://localhost:8081
    http:
      port: 8081
  discovery:
    etcd:
      hosts: http://localhost:2379
    ttl: 20
    ping-interval: 15
  ```

Note: when connecting to Consul, property `kumuluzee.config.etcd.hosts` is ignored as well as property `kumuluzee.server.base-url`, since Consul implementation uses agent's IP address for the URL of registered services.

Port 8081 is used because we want to run another microservice on default port, which discovers this service on port 8080.

### Run microservice

Before you run the sample you should set environmental variables `EXTENSION` and `PORT`:
* EXTENSION: sets configuration source, possible values: `consul` and `etcd`,
* PORT: sets the value of server port, default `8081`.

In the end simply run this command to start microservice:
```
$ npm run start
```

