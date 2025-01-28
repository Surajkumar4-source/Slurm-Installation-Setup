
## Introduction to Slurm Cluster Setup

*Slurm, which stands for Simple Linux Utility for Resource Management, is an open-source, highly scalable workload manager for Linux clusters. It provides job scheduling, resource allocation, job execution, and monitoring capabilities. The Slurm cluster setup typically involves configuring a controller node and multiple compute nodes to efficiently manage and allocate system resources.*

## Overview of the Workflow:

### 1. Configure Hostnames and Networking on the Controller Node:

- The hostnames and IP addresses for the controller and compute nodes are defined to establish a clear network topology. This step ensures proper communication between the nodes.
- Updating /etc/hosts on the controller and setting the hostname helps Slurm recognize the controller node’s role and identify compute nodes.
- Verification through ping commands checks that the networking configuration is correctly set up.
  
### 2. User Configuration:

- A user named slurm is created with a shell and added to the sudo group, allowing it to execute commands with administrative privileges. This user will run Slurm services on both the controller and compute nodes.


### 3. Install Dependencies:

- Essential packages required for building and running Slurm are installed. These include build-essential, munge, libmunge-dev, libmysqlclient-dev, libssl-dev, libpam0g-dev, libnuma-dev, perl, wget, tar, and mariadb-server. These packages provide the necessary tools for compiling and executing Slurm, managing user authentication, and interfacing with the database server.

### 4. Configure Munge:

- Munge is used for user authentication and key distribution across the Slurm cluster. A Munge key is generated and its permissions set to secure communication between nodes.
- The Munge key is distributed to compute nodes to maintain secure access to the Slurm services.

### 5. Database Setup (MariaDB):

- A MariaDB database is set up to store Slurm job accounting data. The slurm_acct_db is created, and user privileges are granted to allow Slurm to perform read/write operations on the database.
- The database is securely configured using mysql_secure_installation.

### 6. Install and Configure Slurm on the Controller Node:

- The Slurm software is downloaded, compiled, and installed on the controller node. Configuration directories and files are created, and necessary adjustments to slurm.conf ensure proper operation of the Slurm daemon (slurmctld).
- slurm.conf is customized to specify the cluster settings, such as the cluster name, host addresses, and authentication method.
- Slurm services (slurmctld, slurmd, slurmdbd) are enabled to start on system boot and manually started for testing.

### 7. Setup cgroup Configuration:

- cgroup.conf is configured to manage resource limits on the compute nodes, ensuring efficient use of CPU, memory, and other resources.

### 8. Configure Slurm Services on Compute Nodes:

- Similar to the controller node, Slurm software is installed on compute nodes. The Munge key and Slurm configuration files are copied from the controller to each compute node.
- The slurmd service is started to manage jobs on compute nodes, and the slurmdbd service is also configured to interface with the database on the controller.

### 9. Verify and Test the Slurm Cluster:

- Basic checks like sinfo and scontrol show nodes confirm the status of the Slurm cluster.
- Test job submission (sbatch) is performed to verify that the cluster can manage and execute jobs correctly.
  


  *This setup forms the backbone of a Slurm-based high-performance computing (HPC) cluster, facilitating efficient job management and resource allocation across multiple nodes.*


<br>
<br>

## Prerequisites for Setting Up a Slurm Cluster

<br>

### 1.System Requirements:

  - OS: Linux (Ubuntu, CentOS, RHEL)
  - Architecture: 32-bit or 64-bit
  - Network: Static IPs, low latency, DNS resolution

### 2.Hardware Requirements:

  - Controller Node: 2 CPUs, 4 GB RAM
  - Compute Nodes: Similar configuration
  - Storage: SSDs for fast I/O

### 3.Software Dependencies:

  - Build Tools: build-essential, gcc, make, wget, tar
  - Munge: For secure authentication
  - Database: MariaDB or MySQL for job data
  -Libraries: libssl-dev, libpam0g-dev, libnuma-dev, libmysqlclient-dev
  - Perl: Required for Slurm scripts

### 4.User Configuration:

  - Slurm User: Create slurm user with sudo privileges
  - Permissions: Ensure proper access to Slurm files
  - Security Configuration:

### 5.Firewall: Open necessary ports (6819, 7003, 7324)
  - SSH Keys: Enable passwordless login
  - Authentication: Configure Munge

### 6.Network Configuration:

  - Hostnames and IPs: Ensure unique hostnames and IPs
  - DNS: Properly configured DNS

### 7.Backup and Monitoring Tools:

  - Backup: Implement backup strategies for job data
  - Monitoring: Use tools like Nagios or Grafana for performance monitoring.


<br>
<br>


# Implementation Setup

<br>
<br>


### Step 1: Configure Hostnames and Networking Controller Node

  - Update /etc/hosts:

```yml
sudo nano /etc/hosts
```
  *Add:*
```yml
192.168.82.124 controller
192.168.82.125 compute1
192.168.82.126 compute2
```
  - Set hostname:

```yml
sudo hostnamectl set-hostname controller
```
  - Verify connectivity:

```yml
ping compute1
ping compute2
```
### Step 2: User Configuration
  - Create a Slurm user:

```yml
sudo useradd -m -s /bin/bash slurm
sudo passwd slurm
```
  - Add user to the sudo group:

```yml
sudo usermod -aG sudo slurm
```
### Step 3: Install Dependencies
  - Install the required packages:

```yml

sudo apt update

sudo apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl wget tar openssh-server mailutils mariadb-server

```
### Step 4: Configure Munge
  - Generate Munge key:

```yml
sudo create-munge-key
```
  - Set permissions:

```yml
sudo chown munge:munge /etc/munge/munge.key

sudo chmod 400 /etc/munge/munge.key

sudo chown -R munge: /etc/munge/ /var/log/munge/

sudo chmod 0700 /etc/munge/ /var/log/munge/

```
  - Start and enable Munge service:

```yml
sudo systemctl start munge

sudo systemctl enable munge

sudo systemctl status munge
```


  - Distribute MUNGE Key to Compute Nodes
    
```yml
# Copy the MUNGE key to compute nodes

sudo scp /etc/munge/munge.key usercontroller@compute:/tmp/

```

  - Start and enable SSH service:

```yml
sudo systemctl start ssh

sudo systemctl enable ssh
```





### Step 5: Database Setup (MariaDB)
  - Install MariaDB:

```yml
sudo apt install mariadb-server   # if missed
```

  - Secure the installation:


```yml
sudo mysql_secure_installation
```

  - Create the Slurm database:

```yml
sudo mysql -u root -p
```
  *Run:*

```yml

CREATE DATABASE slurm_acct_db;

GRANT ALL ON slurm_acct_db.* TO 'slurm'@'localhost' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;

EXIT;

```






### Step 6: Install and Configure Slurm on Controller Node:

  - Download and extract Slurm:

```yml
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2

tar -xvjf slurm-21.08.8.tar.bz2

cd slurm-21.08.8
```

  - Build and install Slurm:

```yml

./configure --prefix=/home/slurm/slurm --with-mysql_config=/usr/bin/mysql_config

make

sudo make install

```

  - Configure Slurm directories:

```yml


sudo mkdir /var/spool/slurmctld

sudo chown slurm:slurm /var/spool/slurmctld

sudo chmod 755 /var/spool/slurmctld


```
Edit slurm.conf:

```yml

# Create SLURM configuration directories
sudo mkdir -p /etc/slurm /etc/slurm-llnl

# Copy and modify SLURM configuration files
cp etc/slurm.conf.example etc/slurm.conf

nano etc/slurm.conf

```
  - Update with:

```yml
ClusterName=cluster
SlurmctldHost=controller
AuthType=auth/munge
StateSaveLocation=/var/spool/slurmctld
SlurmdSpoolDir=/var/spool/slurmd
SlurmUser=slurm
ControlAddr=192.168.82.124
NodeName=compute[1-2] CPUs=1 State=UNKNOWN
PartitionName=default Nodes=ALL Default=YES MaxTime=INFINITE State=UP
MailProg=/usr/bin/mail
```

  - Deploy the configuration
```yml
sudo cp etc/slurm.conf /etc/slurm/

sudo cp etc/slurm.conf /etc/slurm-llnl/
```


### Step 7: Set up cgroup configuration:

```yml
cp etc/cgroup.conf.example etc/cgroup.conf
nano etc/cgroup.conf
```
  *Add*


```yml
touch cgroup.conf
CgroupAutomount=yes
ConstrainCores=no
ConstrainRAMSpace=no
```

### Step 8: Install Slurm services:

```yml

cp slurmctld.service /etc/systemd/system/

cp slurmd.service /etc/systemd/system/

cp slurmdbd.service /etc/systemd/system/

```


### Step 9: Configure environment variables:

```yml
sudo nano /etc/bash.bashrc
```

  *Add:*

```yml
export PATH="/home/usercontroller/slurm-21.08.8/bin/:$PATH"

export PATH="/home/usercontroller/slurm-21.08.8/sbin/:$PATH"

export LD_LIBRARY_PATH="/home/usercontroller/slurm-21.08.8/lib/:$LD_LIBRARY_PATH"

```

  - Enable Slurm services:

```yml
sudo cp etc/slurmctld.service /etc/systemd/system/

sudo systemctl start slurmctld

sudo systemctl enable slurmctld

sudo systemctl status slurmctld

```



### Step 10: Configure SLURM database daemon
```yml
cp etc/slurmdbd.conf.example etc/slurmdbd.conf
nano etc/slurmdbd.conf
```

  *Add*

```yml

# SLURM database daemon configuration file for the database connection and authentication

# Database connection settings
SlurMDBDPort=6819
SlurMDBDHost=localhost  # or controller ip
SlurMDBDUser=slurm
SlurMDBDPass=slurm

# Authentication
AuthType=auth/munge

# Timeouts
SlurMDBDTimeOut=300
SlurMDBDMinJobAge=300

# Paths
StateSaveLocation=/var/lib/slurmdbd

```



  - Set permissions for slurmdbd configuration

```yml
sudo chmod 600 /home/usercontroller/slurm-21.08.8/etc/slurmdbd.conf
```


  - Verify SLURM Setup
```yml

# Check SLURM information
sinfo
```






<br>
<br>
<br>


## Compute Nodes:   (same for all compute node  or clone it after on is setup)

  - Repeat /etc/hosts and hostname setup for each compute node.

  - Copy Munge key and Slurm config: From the controller:

  - On the compute node: (if this step is miss)

```yml
scp /etc/munge/munge.key compute1:/etc/munge/

scp /etc/slurm-llnl/slurm.conf compute1:/etc/slurm-llnl/
```

  - On the compute node:

```yml

sudo chown munge:munge /etc/munge/munge.key

sudo chmod 400 /etc/munge/munge.key
```



  - Download Slurm:

```yml
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
```
  - Install necessary packages:

```yml
sudo apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
```
  - Extract Slurm and change to directory:

```yml
tar -xvjf slurm-21.08.8.tar.bz2
cd slurm-21.08.8
```
  - Configure Slurm installation:

```yml
./configure --prefix=/home/compute/slurm-21.08.8/
make
make install
```
  - Copy Munge key:

```yml
cp -r /tmp/munge.key /etc/munge/

chown -R munge: /etc/munge /var/log/munge

chmod 0700 /etc/munge /var/log/munge

systemctl enable munge

systemctl start munge

systemctl status munge

```

  - Copy Slurm configuration files:

```yml

cp /tmp/slurm.conf /home/compute/slurm-21.08.8/etc/

mkdir /etc/slurm

cp /tmp/slurm.conf /etc/slurm/

mkdir /etc/slurm-llnl

cp /tmp/slurm.conf /etc/slurm-llnl/

cp /tmp/slurm.conf slurm-21.08.8/etc/

```
  - Configure Slurm on compute node:

```yml

cp slurmd.service /etc/systemd/system

scp -r slurm-21.08.8/etc/cgroup.conf compute@controller:/tmp

cp /tmp/cgroup.conf .

service slurmd start

service slurmd status
```

  - Update Slurm configuration:

```yml
systemctl start slurmctld
systemctl status slurmctld
systemctl start slurmdbd
systemctl status slurmdbd
systemctl start slurmd
systemctl status slurmd
```



  *Ensure that all the commands are run as the slurm user or with sudo if required.*




<br>
<br>

  - Test Slurm Cluster


```yml
sinfo

scontrol show nodes
```

  - Submit a test job: Create a job script:

```yml
nano test_job.sh
```
*Add:*

```yml

#!/bin/bash
#SBATCH --job-name=test
#SBATCH --output=test_output.txt
hostname

```
  - Submit the job:

```yml
sbatch test_job.sh
```












<br>

<br>

<br>












## ------------------Screenshots--------------------



![image](https://github.com/user-attachments/assets/ae913ec2-e3d1-4b55-b80e-e81716343918)




![image](https://github.com/user-attachments/assets/237d4628-ee45-4cd3-8103-bc618811918d)


![image](https://github.com/user-attachments/assets/d32a7ce2-2bda-4ebd-b6f1-b49b394c7114)




![image](https://github.com/user-attachments/assets/c1b8c019-98d2-47ce-ba2f-0a32f507e654)


![image](https://github.com/user-attachments/assets/ab0bc6bf-44bd-4daf-bcd0-edb48e67b73e)


![image](https://github.com/user-attachments/assets/a82de9c4-d314-480f-8c17-e5b8ac472eba)



![image](https://github.com/user-attachments/assets/304f6e36-9862-4a77-a259-21f0329d201a)


![image](https://github.com/user-attachments/assets/fc2e85a7-d840-4dbe-9381-54eec66f4ea6)



![image](https://github.com/user-attachments/assets/42fcae3a-12a1-4986-981a-0cc76d60211b)



![image](https://github.com/user-attachments/assets/b42fb67f-ab92-46cd-a347-04fa945a901b)





![image](https://github.com/user-attachments/assets/c57688ca-4220-4b75-96ff-6251e77eb0a5)




![image](https://github.com/user-attachments/assets/2a8ef0ad-71d7-4ba4-843c-f6df1007f137)



![image](https://github.com/user-attachments/assets/4917b6b6-6965-45fc-a30f-25db7be4e977)





![image](https://github.com/user-attachments/assets/d3f8ca63-9f21-4c48-af94-506dac140c64)




![image](https://github.com/user-attachments/assets/c3634340-3772-45b8-b8af-3935d4db4048)




![image](https://github.com/user-attachments/assets/68202d0c-4be7-464e-93cc-cf33c01f313f)



![image](https://github.com/user-attachments/assets/3b392409-d54d-4f15-8c3c-10886c340bdf)


![image](https://github.com/user-attachments/assets/7449eee7-37d1-413c-9d0a-617084289960)


![image](https://github.com/user-attachments/assets/50d42b29-1715-49ef-82ca-2bb1c1dede37)


![image](https://github.com/user-attachments/assets/356e2315-3ed3-42f9-a942-32c9ca0581a5)







![image](https://github.com/user-attachments/assets/e10f38b2-b364-4ad8-aa1d-5275d627f657)


![image](https://github.com/user-attachments/assets/d052767a-51d6-4fa7-bccb-57a2f2c57af4)



![image](https://github.com/user-attachments/assets/d236057d-1b5f-451d-856e-8053229f8894)





![image](https://github.com/user-attachments/assets/5bd0b31e-0ae5-4136-aa3c-0ce252c32e12)




![image](https://github.com/user-attachments/assets/cf27eb01-9994-4e5f-b994-15fb1a3c44f5)




![image](https://github.com/user-attachments/assets/98fbddc1-0370-4bf6-b6ef-dc8baf18f28d)




![image](https://github.com/user-attachments/assets/2112c914-5449-4c8d-b231-0d82c027ad99)






![image](https://github.com/user-attachments/assets/86fbb418-282c-43c3-8103-cd01154028ba)




![image](https://github.com/user-attachments/assets/c6e35eec-069a-4bbf-ad1e-780ffff2ea48)




![image](https://github.com/user-attachments/assets/af3fc55c-ce02-4aec-8e7c-9275856f238d)






![image](https://github.com/user-attachments/assets/af52b0b6-ed04-4b74-bb8e-429da2fb0ee5)












![image](https://github.com/user-attachments/assets/ce11f61a-aef3-40e6-bfca-a27355e4f6ae)





![image](https://github.com/user-attachments/assets/c3d4d732-f85f-4f2f-be37-ff8bac6a4f37)




![image](https://github.com/user-attachments/assets/a299c680-5898-4cf1-9dc4-9c307b7e74a4)





![image](https://github.com/user-attachments/assets/5d26b255-81d8-4980-a479-446dae499607)





![image](https://github.com/user-attachments/assets/8b5c902c-2c72-4fa8-affc-2978a8f9f3f4)



![image](https://github.com/user-attachments/assets/c6195e22-a6eb-41d7-a648-0a489f937c4a)






![image](https://github.com/user-attachments/assets/152c02fd-9447-427e-a3c6-553acd9e8e4e)





![image](https://github.com/user-attachments/assets/6844ca34-c820-4574-a44a-be4f08f2dab5)















<br>
<br>
<br>
<br>



**👨‍💻 𝓒𝓻𝓪𝓯𝓽𝓮𝓭 𝓫𝔂**: [Suraj Kumar Choudhary](https://github.com/Surajkumar4-source) | 📩 **𝓕𝓮𝓮𝓵 𝓯𝓻𝓮𝓮 𝓽𝓸 𝓓𝓜 𝓯𝓸𝓻 𝓪𝓷𝔂 𝓱𝓮𝓵𝓹**: [csuraj982@gmail.com](mailto:csuraj982@gmail.com)





<br>
