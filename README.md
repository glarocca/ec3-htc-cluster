# EC3-htc-cluster

This document describes how to use the Elastic Cloud Computing Cluster (EC3) platform to create elastic virtual clusters on the EGI Cloud.

## Requirements

* Basic knowledge of Linux user environment and [Docker container](https://docs.docker.com/engine/reference/commandline/container/) are requested
* A working environment with the [EC3 Docker CLI](https://github.com/grycap/ec3#ec3-in-docker-hub) installed 
* A valid access token to access the [EGI Cloud infrastructure](https://egi-federated-cloud.readthedocs.io/en/latest/federation.html)

## Install the EC3 CLI

EC3 has an official Docker container image available in Docker Hub that can be used instead of installing the CLI. 
You can download it with:

<pre>
]$ sudo docker pull grycap/ec3
</pre>

The CLI will allow you to exploit all the potential of EC3 from your computer.

## Listing available EC3 templates

To list the available templates, use the command:

<pre>
]$ sudo docker run -v /var/.ec3/clusters:/root/.ec3/clusters grycap/ec3 templates
          name              kind                                         summary                                      
----------------------------------------------------------------------------------------------------------------------
          blcr            component Tool for checkpointing applications.                                              
         bowtie2          component Fast and sensitive read alignment.                                                
       centos-ec2          images   CentOS 6.5 amd64 on EC2.                                                          
         chronos          component An open-source tool to orchestrate job execution in a Mesos cluster               
         ckptman          component Tool to automatically checkpoint applications running on Spot instances.          
         consul           component Service Discovery and Configuration Made Easy                                     
         docker           component An open-source tool to deploy applications inside software containers.            
     docker-compose       component Compose is a tool for defining and running multi-container Docker applications.   
 escherichia_coli_genome  component Download and add the Escherichia_coli_K_12_DH10B NCBI 2008-03-17 genome index in  
        extra_hd          component Adds an extra HD with 100 GB in /mnt directory                                    
  frontend-behind-wall    component Create SSH tunnels from the frontend to the working nodes.                        
         galaxy           component Galaxy is an open, web-based platform for data intensive biomedical research.     
   galaxy-declic-tool     component This component is for automated installation of the INRA Declic tool into Galaxy  
   galaxy-disseq-tool     component This component is for automated installation of the INRA Disseq tool into Galaxy  
    galaxy-glxcc-tool     component This component is for automated installation of the INRA Declic connected compon  
      galaxy-tools        component This component is for automated installation of tools from a Tool Shed into Gala  
         gnuplot          component A program to generate two- and three-dimensional plots.                           
         hadoop           component A framework that allows for the distributed processing of large data sets across  
        htcondor            main    Install and configure a cluster HTCondor from distribution repositories.          
         jupyter          component A tool to share scientific documents.                                             
        kubefaas          component Install FaaS frameworks in a kubernetes cluster                                   
       kubernetes           main    Install and configure a cluster using the grycap.kubernetes ansible role.         
        lemonade          component Live Exploration and Mining Of Non-trivial Amount of Data from Everywhere         
        marathon          component An open-source tool to orchestrate job execution in a Mesos cluster               
          mesos             main    Install and configure a Mesos cluster from mesosphere repo.                       
         monasca          component Monasca is a open-source multi-tenant, highly scalable, performant, fault-tolera  
         mrbayes          component A program for Bayesian inference and model choice across a wide range of phyloge  
         myproxy          component Tool download proxy files from MyProxy servers                                    
      myproxy_ltos        component Tool to refresh LToS proxy.                                                       
          namd            component A parallel, object-oriented molecular dynamics code designed for high-performanc  
           nfs            component Tool to configure shared directories inside a network.                            
          nomad           component Nomad is a tool for managing a cluster of machines and running applications on t  
         octave           component A high-level programming language, primarily intended for numerical computations  
         openvpn          component Tool to create a VPN network.                                                     
           sge              main    Install and configure a cluster SGE from distribution repositories.               
          slurm             main    Install and configure a cluster using the grycap.slurm ansible role.              
       slurm-repo           main    Install and configure a cluster SLURM from distribution repositories.             
          spark           component Lightning-fast cluster computing.                                                 
        swap-disk         component Recipe to add and configure a swap disk to the front-end                          
          swarm             main    Install and configure a Docker Swarm cluster using swarm containers.              
       test-slurm           test    Test ec3_control with SLURM.                                                      
       test-torque          test    Test ec3_control with TORQUE.                                                     
         tomcat           component An open-source web server and servlet container                                   
         torque             main    Install and configure a cluster TORQUE from distribution repositories.            
      ubuntu-azure         images   Ubuntu 12.04 amd64 on Azure.                                                      
       ubuntu-ec2          images   Ubuntu 14.04 amd64 on EC2.                                                        
       ubuntu-fbw          images   Ubuntu 16.04 amd64 on FogBow.                                                     
       ubuntu-gce          images   Ubuntu 14.04 amd64 on Google Cloud.                                               
 ubuntu-hybrid-spot-ec2    images   Example of hybrid cluster with regular and spot instances on EC2.                 
       ubuntu-vmrc         images   Ubuntu 14.04 to search on the VMRC.                                               
        zookeeper         component An open-source server which enables highly reliable distributed coordination  
</pre>

## Listing available EC3 clusters

To list the available running clusters, use the command:

<pre>
]$ sudo docker run -v /var/.ec3/clusters:/root/.ec3/clusters grycap/ec3 list

      name          state           IP        nodes 
----------------------------------------------------
    cluster_1      configured  XXX.XXX.XXX.XXX    0   
    cluster_2      configured  XXX.XXX.XXX.XXX   0   
</pre>

## Authorization file

The authorization file stores in plain text the credentials to access the cloud providers, the IM service and the VMRC service. Each line of the file is composed by pairs of key and value separated by semicolon, and refers to a single credential. The key and value should be separated by `=`, that is an equal sign preceded and followed by one white space at least.

Example of cloud provider with OIDC-based authentication:

<pre>
]$ cat auth.dat
id = PROVIDER_ID; type = OpenStack; host = KEYSTONE_ENDPOINT; username = egi.eu; tenant = openid; domain = DOMAIN_NAME; auth_version = 3.x_oidc_access_token; password = OIDC_ACCESS_TOKEN
</pre>

Where:
* `id` is the id of the cloud provider (e.g.: `in2p3ostegi`)
* `host` is the public IP address of the cloud provider Keystone service (e.g.: `https://sbgcloud.in2p3.fr:5000/v3`)
* `domain` is the project tenant in the cloud provider (e.g.: `EGI_access`)
* `password` is the access token (from EGI AAI Check-In)

## Get an Access Token
Login the EGI AAI Check-In [EGI AAI Check-In](https://aai.egi.eu/fedcloud) service. Copy and paste in your terminal the CURL command to generate a valid access token valid for <b>1h</b>. 

## Template used to configure the EC3 cluster

The cluster will be configured with the following templates:
* torque (default)
* [centos7-OIDC-IN2P3-IRES_Torque (custom)](ec3/templates/centos7-OIDC-IN2P3-IRES_Torque.radl)
* [cluster_configure (custom)](ec3/templates/cluster_configure.radl)
* [configure_nfs (custom)](ec3/templates/configure_nfs.radl)
* [refreshtoken (custom)](ec3/templates/refreshtoken.radl)

User's templates are stored in: `$HOME/ec3/templates`

## Create an elastic EC3 cluster

To launch a cluster, you can use the recipes that you have locally by mounting the folder as a volume, or create your dedicated ones. Also, it is recommendable to maintain the data of active clusters locally, by mounting a volume. In the next example, we are going to deploy a new Torque/Maui cluster in one cloud provider of the EGI Federation. CentOS7 as base OS will be used to configure the front-node and the workers of the EC3 elastic cluster.

<pre>
]$ docker run -v /home/centos/:/tmp/ \
                 -v /home/centos/ec3/templates:/root/.ec3/templates \
                 -v /var/.ec3/clusters:/root/.ec3/clusters grycap/ec3 launch cluster_name \
                 torque centos7-OIDC-IN2P3-IRES_Torque cluster_configure refreshtoken configure_nfs \
                 -a /tmp/auth.dat

Creating infrastructure
Infrastructure successfully created with ID: 529c62ec-343e-11e9-8b1d-300000000002
Front-end state: launching
Front-end state: pending
Front-end state: running
IP: XXX.XXX.XXX.XXX
Front-end configured with IP XXX.XXX.XXX.XXX
Transferring infrastructure
Front-end ready!
</pre>

## Access the EC3 cluster

To access the cluster, use the command:

<pre>
]$ sudo docker run -ti -v /var/.ec3/clusters:/root/.ec3/clusters grycap/ec3 ssh cluster_name
Warning: Permanently added 'XXX.XXX.XXX.XXX' (ECDSA) to the list of known hosts.
Last login: Sat Jul 20 07:14:36 2020 from XXX.XXX.XXX.XXX
[centos@torqueserver ~]$ 
</pre>

## Configuration of the cluster

Check status of the nodes of the cluster:

<pre>
[centos@torqueserver ~]$ clues status
node                          state    enabled   time stable   (cpu,mem) used   (cpu,mem) total
-----------------------------------------------------------------------------------------------
wn1                             off    enabled     00h04'20"      0,0              1,-1
wn2                             off    enabled     00h04'20"      0,0              1,-1
wn3                             off    enabled     00h04'20"      0,0              1,-1
wn4                             off    enabled     00h04'20"      0,0              1,-1
wn5                             off    enabled     00h04'20"      0,0              1,-1
</pre>

Status of the cluster is monitored on regular basis by CLUES, however the user can decide to force the starting/stopping of nodes using the CLI:

<pre>
]$ clues poweron wn1 wn2
node wn1 wn2 powered on
</pre>

The configuration triggers the execution of several ansible processes to configure the node and may take some times.
To monitor the configuration of the node, you can use the `is_cluster_ready` command
Logs files are available in tail -f /var/tmp/.im/...

To avoid that clues poweroff the node in case of inactivities, once the node is configured, you can disable the node
as it follows:

<pre>
[centos@torqueserver ~]$ clues disable wn1 wn2

[centos@torqueserver ~]$ clues status
node                          state    enabled   time stable   (cpu,mem) used   (cpu,mem) total
-----------------------------------------------------------------------------------------------
wn1                             off    disabled     00h04'20"      0,0              1,-1
wn2                             off    disabled     00h04'20"      0,0              1,-1
wn3                             off    enabled      00h04'20"      0,0              1,-1
wn4                             off    enabled      00h04'20"      0,0              1,-1
wn5                             off    enabled      00h04'20"      0,0              1,-1
</pre>

## Enable Password-based authentication
Change settings in `/etc/ssh/sshd_config`

<pre>
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication yes
</pre>

and restart the sshd daemon

<pre>
]# sudo service sshd restart
</pre>

## Testing the cluster
Access the cluster with one of the available account:

<pre>
]$ ssh user01@XXX.XXX.XXX
</pre>

Create a simple test file:
<pre>
]$ cat test.sh
#!/bin/bash
#PBS -N job
#PBS -q batch

#cd $PBS_O_WORKDIR/
hostname -f
sleep 5
</pre>

Submit to the batch queue:
<pre>
]$ qsub -l nodes=2 test.sh
</pre>

## Destroy the EC3 cluster

To destroy a running cluster, use the command:

<pre>
]$ sudo docker run -ti -v /var/.ec3/clusters:/root/.ec3/clusters grycap/ec3 destroy cluster_name
</pre>

## Logs file
* CLUES log file: `/var/log/clues2/clues2.log`
* IM log file: `/var/log/im/im.log`
* Torque/MAUI: `/var/log/torque/server_logs/`

## Additional information

* [EC3 Command-line Interface](http://ec3.readthedocs.org/en/devel/ec3.html)
* [Available EC3 templates](http://ec3.readthedocs.org/en/devel/templates.html)
* Webinar: https://indico.egi.eu/event/5092/

