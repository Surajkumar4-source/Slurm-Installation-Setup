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




