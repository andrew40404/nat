# Installing NATS on IBM Cloud

This document will describe how to install NATS on IBM Cloud using Kubernetes services.

## Step 1 - Provision Kubernetes Cluster

- Click the **Catalog** button on the top
- Select **Service** from the **Catalog**
- Search for **Kubernetes Service** and click on it

![NAT_install on ibm cloud_html_46d1c04e26ba5eea](https://user-images.githubusercontent.com/5286796/106412230-39de1280-646d-11eb-85d8-b48491ff1d12.png)

- You are now at the Kubernetes deployment page. You need to specify some information about the cluster.
- Choose either of the following plans; **standard** or **free**. The free plan only has one worker node and no subnet. To provision a standard cluster. You will need to upgrade your account to Pay-As-You-Go
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

## Step 2 - Deploy IBM Cloud Block Storage plug-in

The Block Storage plug-in is a persistent, high-performance iSCSI storage that you can add to your apps by using Kubernetes Persistent Volumes (PVs).

- Click the **Catalog** button on the top
- Select **Software** from the catalog
- Search for **IBM Cloud Block Storage plug-in** and click on it
  
![mongodb_html_80e526461a17c251](https://user-images.githubusercontent.com/5286796/106396725-c9fd6700-642f-11eb-8606-71998e0bbbc2.png)
   
- On the application page, click on the dot next to the cluster you wish to use
- Click on Enter or Select Namespace and choose the default Namespace or use a custom one (if you get an error please wait 30 minutes for the cluster to finalize)
   
![mongodb_html_c5a3e57a3c6cd652](https://user-images.githubusercontent.com/5286796/106396724-c964d080-642f-11eb-8e55-c82480054778.png)
   
- Give a **name** to this workspace
- Click **install** and wait for the deployment

![mongodb_html_bcca9b451248ae84](https://user-images.githubusercontent.com/5286796/106396722-c79b0d00-642f-11eb-81f9-084f9c9f04be.png)

## Step 3 - Installing NATS

1. Install [Docker](https://docs.docker.com/install)  
2. Install [Helm 	Client](https://helm.sh/docs/using_helm/#installing-the-helm-client) 
3. Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)
4. Install [IBM Cloud CLI](https://cloud.ibm.com/docs/cli/reference/ibmcloud?topic=cloud-cli-install-ibmcloud-cli#shell_install)
5. Install IBM Cloud Kubernetes service (IKS) plugin
6. Login IBM Cloud account
7. Initialize IKS plugin
8. Set kubectl to manage IKS cluster
9. Verify kubectl settings for cluster

### Step 4 Deploy NATS

1. Download NATS Streaming server helm package

```sh
$ wget -O /tmp/nats-ss-0.0.1.tgz https://github.com/ssibm/iks-nats-streaming/raw/master/deploy/nats-ss-0.0.1.tgz
```

2. Create deployment yaml files for NATS Streaming StatefulSet, Service, Persistent Volume, and Persistent Volume Claim. In the command below, persistence will be set to local

```sh
$ mkdir -p /tmp/helm-output
 $ helm template --name test-drive --set persistence.local.enabled=true --output-dir /tmp/helm-output /tmp/nats-ss-0.0.1.tgzOutput:
 wrote /tmp/helm-output/nats-ss/templates/pv.yaml
 wrote /tmp/helm-output/nats-ss/templates/pvc.yaml
 wrote /tmp/helm-output/nats-ss/templates/service.yaml
 wrote /tmp/helm-output/nats-ss/templates/statefulset.yaml
```

3. Deploy NATS Streaming on IKS Cluster using the deployment files

```sh
$ kubectl apply --recursive --filename /tmp/helm-output/nats-ssOutput:
 persistentvolume/pv-nats-ss created
 persistentvolumeclaim/pvc-nats-ss created
 service/test-drive-nats-ss created
 statefulset.apps/test-drive-nats-ss create
```

4. It may take few minutes for the Persistence Volume Claim to finish binding and for the Pods to be in Running state. Verify using the following command

```sh
$ kubectl get all -l release=test-driveNAME READY STATUS RESTARTS AGE
 pod/test-drive-nats-ss-0 1/1 Running 0 6m55sNAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
 service/test-drive-nats-ss LoadBalancer 172.21.108.200 169.46.126.110 4222:30882/TCP,8222:30685/TCP 6m55sNAME DESIRED CURRENT AGE
 statefulset.apps/test-drive-nats-ss 1 1 6m55s
```

5. After the Pods are in the  **Running** state, go to http://<EXTERNAL-IP>:8222 to confirm NATS Streaming monitoring page loads successfully.


The installation is done. Enjoy!

