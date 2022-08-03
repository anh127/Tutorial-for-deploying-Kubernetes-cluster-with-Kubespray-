# Kubespray-Installation-tutorial

# **Set up K8S cluster by using kubespray**

## Turn off swap
    
- Swapping or swap space represents a physical memory page that lives on top of disk partition or a special disk file used for extending the RAM memory of a system when the physical memory fills up.
In case your server has sufficient RAM memory or does not require the use of swap space or the swapping greatly decreases your system performance, you should consider disabling the swap area.

- You can turn off the swap following these command:
  > ` $ sudo swapoff -a `

- After that, open vim **/etc/fstab** (put # in line /swap.img)
  > ` $ sudo vim /etc/fstab `

  ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/3.png)

## Initial setup

First, login as **sudo** user because the following set of commands need to be executed with ‘sudo’ permissions. 
Then, update your ‘apt-get’ repository.

>`$ sudo su`
>`# apt-get update`



To change the hostname of both machines, run the below command to open the file and subsequently rename:

>`# vim /etc/hostname`

Update The Hosts File With IPs Of Master & Node
Run the following command on both machines to note the IP addresses of each.

>`# ifconfig` 

Now go to the ‘hosts’ file on both the master and node and add an entry specifying their respective IP addresses along with their names. This is used for referencing them in the cluster.

>`# vi /etc/hosts`

Then, run `sudo shutdown -r now` to update new configuration.
## Establish SSH connection
  - Create SSH keygen: Generating an SSH key pair creates two long strings of characters: a public and a private key. You can place the public key on any server, and then connect to the server using an SSH client that has access to the private key.
    > `# ssh-keygen -b 4096 -f key`

    - The -f option specifies a file name, foo is an example, use whatever name you wish.

  - You will see the the picture like below ( If you see Overwrite just press y - this means you have already generated the keygen before and now new key will overwrite the old ones):

    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/5.png)
  
### On the server (where you ssh TO)
- Edit the file in `/etc/ssh/ssh_config` to add the following line:
    >` sudo vim /etc/ssh/ssh_config `
- Make sure the line should look like this:

    > PermitRootLogin yes
    > PasswordAuthentication yes

- Change your password ( use the strong one - you should use the same password for both master-node and worker-node):
    >` $ passwd `
    > `$ sudo passwd`
### On the Client (where you ssh FROM)

- Run the following command to copy the public key to the server:
    > `$ ssh-copy-id -i ~/.ssh/key.pub root@IP_ADDRESS`

### On the server (where you ssh TO)

- Change the file in `/etc/ssh/ssh_config`:
    
    > **AllowUsers** root user
    > **PermitRootLogin** without-password
    > **PublicKeyAuthentication** yes
    > **PasswordAuthentication** no

## Install git 

- Installing git by using this command:

  > ` $ sudo apt-get install git`

## Install python 3

- Installing python3 following the command below:

    > ` $ sudo apt-get update `
    > ` $ sudo apt-get install python3.6 `

- To see which version of Python 3 you have installed, open a command prompt and run
  
    > ` $ python3 --version `

## Install pip3
#### Step 1. Update system:

-   > ` $ sudo apt-get update`

#### Step 2 - Install pip3

- If Python 3 has already been installed on the system, execute the command below to install pip3:
    > ` $ sudo apt-get -y install python3-pip`

#### Step 3 - Verification

-    > ` $ pip3 --version `

## Installing Kubespray 

- Git clone the kubespray by following this command ( I use version 18.1 but you can use whatever version you like):

    > ` git clone https://github.com/kubernetes-sigs/kubespray.git `

- Install dependencies from `requirements.txt`    
    
    >  `$ cd kubespray/`
- It is recommended to deploy the ansible version used by kubespray into a python virtual environment.
    >  `$ VENVDIR=kubespray-venv`

    > ` $ KUBESPRAYDIR=kubespray`

    >   `$ ANSIBLE_VERSION=2.12`

    >   `$virtualenv  --python=$(which python3) $VENVDIR`

    > `$    source $VENVDIR/bin/activate`

    > ` $ pip install -U -r requirements-$ANSIBLE_VERSION.txt`
    > `test -f requirements-$ANSIBLE_VERSION.yml && \
  ansible-galaxy role install -r requirements-$ANSIBLE_VERSION.yml && \
  ansible-galaxy collection -r requirements-$ANSIBLE_VERSION.yml`

### Ansible

#### Usage

   - > ` $ sudo pip3 install -r requirements.txt `

***Optional:*** 
If you plan to change the CNI plugin from the default(calico) to something else like Weave then modify the following file, This is an optional step,  I have performed this as I wanted a cluster with weave installed.  At this point if you want to change the version to something else then make the change. 
 > ` $  vim inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml` 
 > `# Choose network plugin (cilium, calico, weave or flannel. Use cni for generic cni plugin)`
> `# Can also be set to 'cloud', which lets the cloud provider setup appropriate routing`
**kube_network_plugin: weave** 
   
- **Warning** : You may see this error:
    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/4.png)
        
    - Allow prerelease `gevent` versions via:

        > ` $ pip install gevent --pre `
        > ` $ pip install auto-py-to-exe `

        Explanation: `auto-py-to-exe` is installable on Python 3.8 on Windows without any issues (this can be verified e.g. by running pip install auto-py-to-exe --no-deps). However, it requires bottle-websocket to be installed, which in turn has `gevent` dependency. gevent did not release a stable version that offers prebuilt wheels for Python 3.8 yet (this would be 1.5), so pip doesn't pick up prebuilt wheels and tries to build gevent == 1.4 from source dist. Installing the prerelease 1.5 version of gevent avoids this.
    
- Copy ``inventory/sample`` as ``inventory/mycluster``

    > ` # cp -rfp inventory/sample inventory/mycluster `

- Update Ansible inventory file with inventory builder

   > ` # declare -a IPS=(IP-address-of-master IP-address-of-worker) `
   > ` # CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}`

   - You may see this error:
        
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/6.png)
    
    -  This error can be fixed by applying the below command:
     > ` # pip3 install ruamel.yaml `

- After finish updating Ansible inventory, you should see result like this ( my cluster has one worker node and one master node)):
    
    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/7.png)

- Check file host.yaml
    > ` vim inventory/mycluster/hosts.yaml `
    
    -You can see the result something like this:
    
    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/.png)
    
        - However, I intend to use node 1 as control plane aka master node and node 2 as worker node. Therefore, I will delete node 2 in kube_control_plane:


    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/9.png)


- Review and change parameters under ``inventory/mycluster/group_vars``

    > ` # vim inventory/mycluster/group_vars/all/all.yml `
    > ` # vim inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml `

- Deploy Kubespray with Ansible Playbook - run the playbook as root (if you deploy ansible playbook in master node you should add **-kK**)
The option `--become` is required, as for example writing SSL keys in /etc/, installing packages and interacting with various systemd daemons.
Without `--become` the playbook will fail to run!
    > ` $ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml -kK`

## Start Kubeadm

- Try the below command:
    > ` $ kubectl get nodes `
    - You may see this error:
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/10.png)

- Fix this error by following the below command:

    > ` $ mkdir -p $HOME/.kube`
    > ` $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
    > ` $ sudo chown $(id -u):$(id -g) $HOME/.kube/config`

- Try the command above again:

    > ` $ kubectl get nodes `

    - It should look like this:
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/11.png)
    
    > ` $ kubectl get pods -A`

    - It should look like this:
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/12.png)

- Look carefully in the red box you can see that the `core-dns-5f44f89dcc-plrft` falls to **Pending State** looping forever.
    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/13.png)
    - To fix this problem, run those commands:
        > ` $ kubectl describe pods core-dns-5f44f89dcc-plrft -n kube-system `
    
    - The picture below describes the problem:

    ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/14.png)

    - The problem is that the pod is trying to deploy into the master node which has tainted mark ( the mark from the developer tainted it so that no pods can be deployed by **Scheduler** into it)

    - You can fix this by running the commands below (the syntax **-** means untaint):
        > ` $ kubectl get nodes `
        > ` $ kubectl taint nodes --all node-role.kubernetes.io/master- `

        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/11.png)
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/16.png)
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/15.png)
    
    - And there you go, the pod is now able to deploy into the master node.
        ![](https://github.com/TranAnh-Tuan/Kubespray-Installation-tutorial/blob/main/images/17.png)

    **Warning** : Remmember to taint the master node immediatly after you have finished the deployment.
    - > ` $ kubectl taint nodes kube-master1 node.kubernetes.io/kube-master1:NoSchedule `

    ### Author
    ---
    ##### Name: Ngọc Ánh Phạm
    ##### Email: phamngocanh2711@gmail.com
    ---
    ##### Name: Trần Anh Tuấn
    ##### Email: tuan-hs11115@ngoisao.edu.vn
