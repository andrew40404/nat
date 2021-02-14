# Installing NATS on IBM Cloud

This document will describe how to install NATS on IBM Cloud using Kubernetes services.

## Step 1 - Provision Kubernetes Cluster

- Click the **Catalog** button on the top
- Select **Service** from the **Catalog**
- Search for **Kubernetes Service** and click on it

![NAT_install on ibm cloud_html_46d1c04e26ba5eea](https://user-images.githubusercontent.com/5286796/106412230-39de1280-646d-11eb-85d8-b48491ff1d12.png)

- You are now at the Kubernetes deployment page. You need to specify some information about the cluster.
- Choose either of the following plans; **standard** or **free**. The free plan only have one worker node and no subnet. To provision a standard cluster. You will need to upgrade your account to Pay-As-You-Go
- To upgrade to a Pay-As-You-Go account, complete the following steps:
- In the console, go to Manage > Account.
- Select Account settings and click `Add credit card`.
- Enter your payment information, click Next, and submit your information
- Choose **classic** or **VPC** , read the docs and choose the most suitable type for yourself

  ![NAT_install on ibm cloud_html_4d3a968071544952](https://user-images.githubusercontent.com/5286796/106412226-39457c00-646d-11eb-9ad0-5055ca1a9ae2.png)

- Now choose your location settings,
- Choose **Geography** (continent)

![NAT_install on ibm cloud_html_72496e6b0b2c820d](https://user-images.githubusercontent.com/5286796/106412223-36e32200-646d-11eb-822b-4abe72488d00.png)

- Choose Single or Multizone. 

> In single zone, your data is only kept on the datacenter while on the other hand with Multizone, it is distributed to multiple zones, thus safer in an unforeseen zone failure
>
> If you wish to use Multizone, please set up your account with VRF
> 

- If at your current location selection, there is no available Virtual LAN, a new VLAN will be created for you
- Choose a Worker node setup or use the preselected one. Set Worker node amount per zone
- Choose **Master Service Endpoint**. 

> In VRF-enabled accounts, you can choose private-only to make your master accessible on the private network or via VPN tunnel. Choose public-only to make your master publicly accessible. When you have a VRF-enabled account, your cluster is set up by default to use both private and public endpoints.
   
- Give desired **tags** to your cluster, click **create**
- Wait for your cluster to be provisioned
- Your cluster is ready for usage

## Step 2 Deploy IBM Cloud Block Storage plug-in

The Block Storage plug-in is a persistent, high-performance iSCSI storage that you can add to your apps by using Kubernetes Persistent Volumes (PVs).

- Click the **Catalog** button on the top
- Select **Software** from the catalog
- Search for **IBM Cloud Block Storage plug-in** and click on it
  
![mongodb_html_80e526461a17c251](https://user-images.githubusercontent.com/5286796/106396725-c9fd6700-642f-11eb-8606-71998e0bbbc2.png)
   
- On the application page, click in the dot next to the cluster you wish to use
- Click on Enter or Select Namespace and choose the default Namespace or use a custom one (if you get error please wait 30 minutes for the cluster to finalize)
   
![mongodb_html_c5a3e57a3c6cd652](https://user-images.githubusercontent.com/5286796/106396724-c964d080-642f-11eb-8e55-c82480054778.png)
   
- Give a **name** to this workspace
- Click **install** and wait for the deployment

![mongodb_html_bcca9b451248ae84](https://user-images.githubusercontent.com/5286796/106396722-c79b0d00-642f-11eb-81f9-084f9c9f04be.png)

## Step 3 Installing NATS

### Prerequisites

- IBM Cloud Block Storage plug-in  
- Kubernetes 1.12+
- Helm 3.1.0
- PV provisioner support in the underlying infrastructure


### Installation 

To install the chart with the release name my-release:

```sh
$ helm install my-release bitnami/mongodb
```
The command deploys MongoDB on the Kubernetes cluster in the default configuration. The [Parameters](https://hub.kubeapps.com/#parameters) section lists the parameters that can be configured during installation.

### Initialize a fresh instance

The [Bitnami MongoDB](https://github.com/bitnami/bitnami-docker-mongodb) image allows you to use your custom scripts to initialize a fresh instance. In order to execute the scripts, you can specify them using the initdbScripts parameter as dict.

You can also set an external ConfigMap with all the initialization scripts. This is done by setting the initdbScriptsConfigMap parameter. Note that this will override the previous option.

The allowed extensions are .sh, and .js.

### Accessing MongoDB nodes from outside the cluster

In order to access MongoDB nodes from outside the cluster when using a replica set architecture, a specific service per MongoDB pod will be created. 

There are two ways of configuring external access:

- Using LoadBalancer services
- Using NodePort services

### Using loadbalancer services

There are two alternatives to use LoadBalancer services:

1. Use random load balancer IPs using an **initContainer** that waits for the IPs to be ready and discover them automatically

```yaml
architecture=replicaset
replicaCount=2
externalAccess.enabled=true
externalAccess.service.type=LoadBalancer
externalAccess.service.port=27017
externalAccess.autoDiscovery.enabled=true
serviceAccount.create=true
rbac.create=true
```
> Note: This option requires creating RBAC rules on clusters where RBAC policies are enabled.

2.  Manually specify the load balancer IPs:

```sh
architecture=replicaset
replicaCount=2
externalAccess.enabled=true
externalAccess.service.type=LoadBalancer
externalAccess.service.port=27017
externalAccess.service.loadBalancerIPs[0]='external-ip-1'
externalAccess.service.loadBalancerIPs[1]='external-ip-2'}
```

> Note: You should know the load balancer IP’s in advance so that each MongoDB node advertised hostname is configured with it.

### Using nodeport services

Manually specify the node ports to use:

```sh
architecture=replicaset
replicaCount=2
externalAccess.enabled=true
externalAccess.service.type=NodePort
externalAccess.service.nodePorts[0]='node-port-1'
externalAccess.service.nodePorts[1]='node-port-2'
```

> Note: You need to know in advance the node ports that will be exposed so each MongoDB node advertised hostname is configured with it.

### Persistence

The [Bitnami MongoDB](https://github.com/bitnami/bitnami-docker-mongodb) image stores the MongoDB data and configurations at the /bitnami/mongodb path of the container.

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) at this location. The volume is created using dynamic volume provisioning.

### Adjust permissions of persistent volume mountpoint

As the image runs as non-root by default, it is necessary to adjust the ownership of the persistent volume so that the container can write data into it. By default, the chart is configured to use Kubernetes Security Context to automatically change the ownership of the volume. However, this feature does not work in all Kubernetes distributions.

As an alternative, this chart supports using an initContainer to change the ownership of the volume before mounting it in the final destination. You can enable this initContainer by setting volumePermissions.enabled to true.

### Using Prometheus rules

You can use custom Prometheus rules for Prometheus operator by using the prometheusRule parameter, see below a basic configuration example:

metrics:

```yaml
enabled: true
prometheusRule:
  enabled: true
  rules:
  - name: rule1
  rules:
  alert: HighRequestLatency
  expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
  for: 10m
  labels:
  severity: page
  annotations:
  summary: High request latency
```

### Enabling SSL/TLS

This container supports enabling SSL/TLS between nodes in the cluster, as well as between mongo clients and nodes, by setting the MONGODB_EXTRA_FLAGS and MONGODB_CLIENT_EXTRA_FLAGS environment variables, together with the correct MONGODB_ADVERTISED_HOSTNAME. To enable full TLS encryption set tls.enabled to true

### Using your own CA

To use your own CA set tls.caCert and tls.caKey with appropriate base64 encoded data. The secrets-ca.yaml will utilise this data to create secret. 

### Accessing the cluster 

To access the cluster you will need to enable the initContainer which generates the MongoDB server/client pem needed to access the cluster. Please ensure that you include the $my_hostname section with your actual hostname and the alternative hostnames section should contain the hostnames you want to allow access to the MongoDB replica set.

### Starting the cluster

After the certs have been generated and made available to the containers at the correct mount points, the MongoDB server will be started with TLS enabled. The options for the TLS mode will be (disabled|allowTLS|preferTLS|requireTLS). This value can be changed via the MONGODB_EXTRA_FLAGS field using the tlsMode. The client should now be able to connect to the TLS enabled cluster with the provided certs.

### Setting Pod's affinity

This chart allows you to set your custom affinity using the XXX.affinity parameter(s). Find more information about Pod's affinity in the [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

As an alternative, you can use the preset configurations for pod affinity, pod anti-affinity, and node affinity available at the [bitnami/common](https://github.com/bitnami/charts/tree/master/bitnami/common#affinities) chart. To do so, set the XXX.podAffinityPreset, 

The installation is done. Enjoy!

