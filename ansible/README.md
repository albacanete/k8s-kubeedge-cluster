# Ansible
The Ansible folder contains the *playbooks* and *roles* that emulate the bash scripts.  

## Inventories
An inventory is a configuration file that defines the hosts and groups of hosts on which Ansible operations should be performed. 

It allows us to define a first version of the Compute Continuum as a Kubernetes cluster. The `jetsons-inventory.yaml` file classifies three types of machines:
- Master: master of the Kubernetes cluster. It will also run the KubeEdge master (cloudcore). 
- Cloud: nodes that will be a Kubernetes node. 
- Edge: nodes that will be a KubEdge node (edgecore).  

Official guide on how to build an inventory: https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

## Roles
Roles are a way to organize and package related tasks, variables, and files into reusable units. They provide a structured approach to breaking down complex Ansible playbooks into smaller, more manageable components. They allow to achieve modularization and code reusability and will be used by one or more playbooks.

We define roles for:
- Installing Kubernetes dependencies on nodes (e.g container runtime and container networking).
- Installing Kubernetes. 
- Installing KubeEdge as binary.
- Installing KubeEdge with `keadm`. 
- Creating a Kubernetes cluster.
- Deploying the KubeEdge master as binary. 
- Deploygin the KubeEdge master as Kubernetes pod.
- Joining cloud node.
- Joining edge node.  
- Deleting a Kubernetes cluster. 

## Playbooks
Playbooks are configuration files written in YAML format that define a set of tasks and configurations to be executed on the remote hosts defined in inventories. In our case, they are composed of different roles. 

We have defined to types of playbooks:
- Installation
    - `master-install.yaml`. We assume that both masters, Kubernetes and KubeEdge, will run on the same machine.
    - `node-cloud-install.yaml`
    - `node-edge-install.yaml`
- Deployment
    - `delete-cluster.yaml`
    - `k8s-create-cluster.yaml`
    - `k8s-join-cluster.yaml`
    - `kubeedge-deploy-master.yaml`
    - `kubeedge-join-cluster.yaml`

## Execution
In order to create or restart a cluster, you should run the commands explained in this sections. You should run them inside the `PROJECT_FOLDER/ansible` folder. 


> First, export the `ANSIBLE_ROLES_PATH`. 
>> You need to execute the following command if your ansible folder is not in the default Ansible home. 
```
PROJECT_FOLDER=/home/acanete/Documentos/kubernetes/k8s-kubeedge-install
export ANSIBLE_ROLES_PATH=$PROJECT_LOCAL_FOLDER/ansible/roles/
```
### Delete cluster
1. Run the following command to delete any existing cluster. Set `$USER` to your jetsons' user. 
```
ansible-playbook -i inventories/jetsons-inventory.yaml playbooks/delete-cluster.yaml -u $USER --ask-become-pass
```

### Create cluster
1. First, create a Kubernetes cluster with the master node set in the `jetsons-inventory.yaml` file. Set `$USER` to your jetsons' user. 
```
ansible-playbook -i inventories/jetsons-inventory.yaml playbooks/kubernetes-create-cluster.yaml -u $USER --ask-become-pass
```
2. Add the cloud nodes specified in the `jetsons-inventory.yaml` file. Set `$USER` to your jetsons' user. 
```
ansible-playbook -i inventories/jetsons-inventory.yaml playbooks/kubernetes-join-cluster.yaml -u $USER --ask-become-pass
```
3. Deploy the KubeEdge master. Set `$USER` to your jetsons' user. 
```
ansible-playbook -i inventories/jetsons-inventory.yaml playbooks/kubeedge-deploy-master.yaml -u $USER --ask-become-pass
```
4. Add the edge nodes specified in the `jetsons-inventory.yaml` file. Set `$USER` to your jetsons' user. 
```
ansible-playbook -i inventories/jetsons-inventory.yaml playbooks/kubeedge-join-cluster.yaml -u $USER --ask-become-pass
```
