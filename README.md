# Slurm-Installation-Setup







To Organize the Steps for Setting Up a Slurm Multi-Node Installation Properly, here's the sequence for setting up the controller and compute nodes:
Step 1: Host Configuration (Controller Node)
Create a host-only network if possible.
Create the machines:
Open /etc/hosts:
bash
Copy code
nano /etc/hosts
192.168.82.124 controller
Set the hostname:
bash
Copy code
nano /etc/hostname
controller
Add a user:
bash
Copy code
useradd usercontroller
passwd usercontroller
usermod -s /bin/bash usercontroller
su - usercontroller
mkdir /home/usercontroller
chown usercontroller:usercontroller /home/usercontroller
chmod 700 /home/usercontroller
Configure sudoers file:
bash
Copy code
nano /etc/sudoers
usercontroller ALL
Step 2: Slurm Multi-node Installation (Controller Node)
Download Slurm:
bash
Copy code
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
Install necessary packages:
bash
Copy code
sudo apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
Extract Slurm and change to directory:
bash
Copy code
tar -xvjf slurm-21.08.8.tar.bz2
cd slurm-21.08.8
Configure Slurm installation:
bash
Copy code
./configure --prefix=/home/controller/slurm-21.08.8/
make
make install
Create and configure Munge key:
bash
Copy code
create-munge-key
chown munge: /etc/munge/munge.key
chmod 400 /etc/munge/munge.key
Install and configure SSH:
bash
Copy code
apt-get install ssh
ufw allow ssh
Secure the Munge service:
bash
Copy code
chown -R munge: /etc/munge /var/log/munge
chmod 0700 /etc/munge /var/log/munge
systemctl enable munge
systemctl start munge
Create Slurm configuration directories:
bash
Copy code
sudo mkdir /etc/slurm-llnl
cd etc
cp slurm.conf.example /etc/slurm-llnl/slurm.conf
nano /etc/slurm-llnl/slurm.conf
Configure slurm.conf:
ClusterName=cluster
SlurmctldHost=controller
AuthType=auth/munge
SlurmUser=controller
NodeName=controller CPUs=1 State=UNKNOWN
NodeName=compute CPUs=1 State=UNKNOWN
PartitionName=partition Nodes=ALL Default=YES MaxTime=INFINITE State=up
MailProg=/usr/bin/mail
bash
Copy code
apt-get install mailutils
Set up cgroup configuration:
bash
Copy code
touch cgroup.conf
CgroupAutomount=yes
ConstrainCores=no
ConstrainRAMSpace=no
Move Slurm configuration to /etc:
bash
Copy code
mkdir /etc/slurm
cp /etc/slurm-llnl/slurm.conf /etc/slurm/
Install Slurm services:
bash
Copy code
cp slurmctld.service /etc/systemd/system/
cp slurmd.service /etc/systemd/system/
cp slurmdbd.service /etc/systemd/system/
mkdir /var/spool/slurmctld
Modify PATH and LD_LIBRARY_PATH:
bash
Copy code
nano /etc/bash.bashrc
export PATH="/home/controller/slurm-21.08.8/bin/:$PATH"
export LD_LIBRARY_PATH="/home/controller/slurm-21.08.8/lib/:$LD_LIBRARY_PATH"
Step 3: Slurm Multi-node Installation (Compute Node)
Prepare the compute node:
Download Slurm:
bash
Copy code
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
Install necessary packages:
bash
Copy code
apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
Extract Slurm and change to directory:
bash
Copy code
tar -xvjf slurm-21.08.8.tar.bz2
cd slurm-21.08.8
Configure Slurm installation:
bash
Copy code
./configure --prefix=/home/compute/slurm-21.08.8/
make
make install
Copy Munge key:
bash
Copy code
cp -r /tmp/munge.key /etc/munge/
chown -R munge: /etc/munge /var/log/munge
chmod 0700 /etc/munge /var/log/munge
systemctl enable munge
systemctl start munge
systemctl status munge
Copy Slurm configuration files:
bash
Copy code
cp /tmp/slurm.conf /home/compute/slurm-21.08.8/etc/
mkdir /etc/slurm
cp /tmp/slurm.conf /etc/slurm/
mkdir /etc/slurm-llnl
cp /tmp/slurm.conf /etc/slurm-llnl/
cp /tmp/slurm.conf slurm-21.08.8/etc/
systemctl stop ufw
iptables -F
Configure Slurm on compute node:
bash
Copy code
cp slurmd.service /etc/systemd/system
scp -r slurm-21.08.8/etc/cgroup.conf compute@compute:/tmp
cp /tmp/cgroup.conf .
service slurmd start
service slurmd status
Update Slurm configuration:
bash
Copy code
nano slurm.conf
systemctl start slurmctld
systemctl status slurmctld
systemctl start slurmdbd
systemctl status slurmdbd
systemctl start slurmd
systemctl status slurmd
