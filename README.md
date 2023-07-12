# Aether-On-Ramp
This repo builds a standalone Aether core with a physical RAN setup, AMF and gNbsimulation.

To download the Aether-On-Ramp repository, use the following command:
```
git clone --recursive https://gitlab.com/onf-internship/aetheronramp.git
```
It build upen following repositories 

## 5gc

The 5gc repository builds a standalone Aether core with a physical RAN setup. It utilizes the k8 repository as a submodule to create a multi-node cluster and run the 5g-aether core on top.

To download the 5gc repository, use the following command:
```
git clone --recursive https://gitlab.com/onf-internship/5gc.git
```

### Step-by-Step Installation
To install the 5g-core, follow these steps:
1. Set the configuration variables in the vars/main.yaml file.
   - Set the "standalone" parameter to deploy the core independently from roc.
   - Specify the "data_iface" parameter as the network interface name of the machine.
   - Set the "values_file" parameter:
     - Use "hpa-5g-values.yaml" for a stateless 5g core.
     - Use "nohpa-5g-values.yaml" for a stateful 5g core.
   - The "custom_ran_subnet" parameter if left empty, core will use the subnet of "data_iface" for UPF.
2. Add the hosts to the init file.
3. Run `make ansible`.
4. In the running Ansible docker terminal, run `make 5gc-install`.
   - This command installs the k8 cluster using `k8s-install`.
   - It creates networking interfaces for UPF, such as access/core, using `5gc-router-install`.
   - Finally, it installs the 5g core using the values specified in `5gc-core-install`.
     - The installation process may take up to 3 minutes.

#### One-Step Installation
To install 5gc in one go, run `make 5gc-install`.
#### Uninstall
   - run `make 5gc-install`


## AMP - Aether Management Platform

The amp repository builds Aether Management Platform that has Aether-ROC and Monitoring system. It utilizes the k8 repository as a submodule to create a multi-node cluster.

To download the AMP repository, use the following command:
```
git clone --recursive https://gitlab.com/onf-internship/amp.git
```
### Step-by-Step Installation
To install the amp, follow these steps:
1. Install k8 cluster
   - As AMF uses K8 submodule, install k8 cluster using `make k8s-install`
2. Install ROC
   - Specify helm charts for `atomix, onosproject, aether_roc`
   - Run `roc-install`.
   - Run `make 5g-roc-install` for 5G or `make 4g-roc-install` for 5G 
3. Install Monitering
   - Specify helm charts for `monitor, monitor-crd`
   - Run `monitor-install`.

#### One-Step Installation
To install AMF in one go, run `amp-install`.
#### Uninstall
   - run `make monitor-uninstall; make roc-uninstall`


## gNbsim

The gnbsim repository enables the building of a multi-container and multi-node cluster of gNbsim using Docker. This allows you to run gNbsim simulations on multiple VMs, with the flexibility to run multiple containers on each VM.

To download the gnbsim repository, use the following command:
```
git clone https://gitlab.com/onf-internship/gnbsim.git
```

### Step-by-Step Installation
To install gnbsim, follow these steps:

1. Install Docker by running `make gnbsim-docker-install`.
2. Configure the network for gnbsim:
   - Set the "data_iface" parameter to the network interface name of the machine.
   - Set "mcavlan_iface" to the name of the macvlan interface to be created.
   - Set "macvlan_network_name" to the name of the Docker network to be created.
   - Set "subnet_prefix" to the first two bytes of the subnet, which should correspond to the "custome_ran_subnet" of 5g-core or the machine's subnet.
3. Start the gNbsim Docker containers using `make gnbsim-docker-start`:
   - Set the container "image" for gNbsim.
   - Set "prefix" to the desired name for the gNbsim containers.
   - Set "count" to the number of containers to be instantiated on each VM.
4. Start the simulation:
   - Set "amf.ip" to the IP address of the core machine.
   - Set "ueid_base" to the starting IMSI number.
     - Each container will have a different starting IMSI number.
   - Set "ue_per_pod" to the number of UEs on each gNbsim container.
   - Set the "gnbsims" array with a list of key-value pairs:
     - Each key represents the container number (numeric).
     - The corresponding value `config_file` is the config template file for gNbsim.
   - Run `make gnbsim-simulator-start`.
5. Check the results:
   - Enter one of the Docker containers using `docker exec -it *prefix*-1 bash`.
   - Use `cat summary.log` to view the last result.
   - To check the logs generated by gnbsim, use `cat *gnbsim*.log`.

#### One-Step Installation
To install gnbsim in one go, run `make gnbsim-simulator-setup-install`.

#### Uninstall
To uninstall gnbsim, run `make gnbsim-simulator-setup-uninstall`.    


## K8:

The K8 repository builds a multi-node K8 cluster using rke2 and installs Helm. To set up the K8 repository, you need to provide the following:

1. Node configurations with IP addresses in the host.ini file.
   - You can specify both master and worker nodes.

2. rke2 configuration parameters, such as the rke2 version, in the ./var/main.yaml file.
3. Rke2 cluster parameters file in config folder
4. run `make k8s-install`

The repository will build a multi-node cluster. To check the cluster, run the following command on the master node:
```sudo /var/lib/rancher/rke2/bin/kubectl get nodes --kubeconfig /etc/rancher/rke2/rke2.yaml```

#### Useful commands:
1. `kubectl get nodes`
2. `sudo /var/lib/rancher/rke2/bin/kubectl get nodes --kubeconfig /etc/rancher/rke2/rke2.yaml`

#### Uninstall
To destroy k8 cluster use `make k8s-uninstall`

--- 

## to Make it multiNode to single Node
1. Destroy cluster using `make aether-uninstall`
2. update host.ini file 
3. Deploy cluster using `5gc-install` 
4. Setup gNbSim
    a. Check amf ip address is set to core ip adddes
    b. Set SameMachineAsCore=true
    c. Set "subnet_prefix" to the first two bytes of the subnet, which should correspond to the "custome_ran_subnet" of 5g-core or the machine's subnet.
    d. Run `gnbsim-simulator-setup-install`

### To setup Gnbsim on same machine as Core
 set SameMachineAsCore = true
 make sure gnbsim.subnet_prefix has same prefix value core.custome_ran_subnet

### Add a node to cluster
TODO
### Remove Node from cluster
TODO
### Add more gNbsim container
TODO

