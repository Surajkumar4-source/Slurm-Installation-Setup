## creae a host only network if possible
## create a machines 
## change the hostname
```bash
nano /etc/hosts
192.168.82.124 controller
nano /etc/hostname
controller
useradd usercontroller 
passwd usercontroller

usermod -s /bin/bash usercontroller
su - usercontroller 
mkdir /home/usercontroller 
chown usercontroller:usercontroller /home/usercontroller
chmod 700 /home/usercontroller


nano /etc/sudoers 
usercontroller ALL
```
# Slurm Multi node Installation-controller
```bash
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
```

```bash
sudo apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
```

```bash
tar -xvjf slurm-21.08.8.tar.bz2  slurm-21.08.8/
```

```bash
cd slurm-21.08.8
```
```bash
./configure --prefix=/home/controller/slurm-21.08.8/
```
```bash
make
```
```bash 
make install
```
```bash 
create-munge-key
```
```bash 
chown munge: /etc/munge/munge.key
```
```bash 
chmod 400 /etc/munge/munge.key 
```
```bash 
apt-get install ssh
ufw allow ssh
```
```bash
scp -r /etc/munge/munge.key compute@compute:/tmp
```
```bash
chown -R munge: /etc/munge /var/log/munge
```
```bash 
chmod 0700 /etc/munge /var/log/munge
```
```bash 
systemctl enable munge
```
```bash 
systemctl start munge
```
```bash
sudo mkdir /etc/slurm-llnl 
```
```bash
cd etc 
cp slurm.conf.example /etc/slurm-llnl/slurm.conf
```
```bash
nano /etc/slurm-llnl/slurm.conf
```
```bash
ClusterName=cluster
SlurmctldHost=controller
AuthType=auth/munge


SlurmUser=controller

comment this->#JobAcctGatherType=jobacct_gather/none

AccountingStorageType=accounting_storage/slurmdbd

NodeName=controller CPUs=1 State=UNKNOWN 
NodeName=compute CPUs=1 State=UNKNOWN
PartitionName=partition Nodes=ALL Default=YES MaxTime=INFINITE State=up 
MailProg=/usr/bin/mail
```
```bash 
apt-get install mailutils
```

in slurm etc 
```bash
touch cgroup.conf
CgroupAutomount=yes 
ConstrainCores=no 
ConstrainRAMSpace=no
```

```bash
mkdir /etc/slurm
```
```bash 
cp /etc/slurm-llnl/slurm.conf /etc/slurm/
```
```bash
scp /etc/slurm-llnl/slurm.conf compute@compute:/tmp
```
```bash
cp slurmdbd.conf.example etc/slurmdbd.conf
```
nano slurmdbd.conf 
```bash
AuthType=auth/munge

DbdAddr=10.1.1.21
DbdHost=Controller

SlurmUser=vboxuser

DebugLevel=verbose

LogFile=/var/log/slurm/slurmdbd.log
PidFile=/var/run/slurmdbd.pid

StorageType=accounting_storage/mysql

StoragePass=vaibhav07
StorageUser=vboxuser
```


```bash
apt-get install mariadb-server
```
```bash
mysql -u root -p 
```
```bash
CREATE USER 'root'@'controller' identified by 'passowrd';
CREATE USER 'root'@'localhost' identified by 'passowrd';
CREATE USER 'root'@'%' identified by 'passowrd';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'controller';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';

```
```bash
cp slurmctld.service /etc/systemd/system/
```
```bash 
cp slurmd.service /etc/systemd/system/
```
```bash
cp slurmdbd.service /etc/systemd/system/
```
```bash
mkdir /var/spool/slurmctld
```
```bash
nano /etc/bash.bashrc
export PATH="/home/root/slurm-21.08.8/bin/:$PATH"
export PATH="/home/root/slurm-21.08.8/sbin/:$PATH"
export LD_LIBRARY_PATH="/home/root/slurm-21.08.8/lib/:$LD_LIBRARY_PATH"
```
# Compute 
```bash
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
```
```bash
apt install -y build-essential munge libmunge-dev libmunge2 libmysqlclient-dev libssl-dev libpam0g-dev libnuma-dev perl
```
```bash
tar -xvjf slurm-21.08.8.tar.bz2
``` 
```bash
cd slurm-21.08.8
```

```bash
./configure --prefix=/home/compute/slurm-21.08.8/
```

```bash
make
```
```bash
make install
```

```bash
cp -r /tmp/munge.key /etc/munge/
```
```bash
chown -R munge: /etc/munge /var/log/munge
```

```bash
chmod 0700 /etc/munge /var/log/munge
```
```bash
systemctl enable munge
systemctl start munge
systemctl status munge
```
```bash
cp /tmp/slurm.conf /home/compute/slurm-21.08.8/etc/
```
```bash
mkdir /etc/slurm
```
```bash
cp /tmp/slurm.conf /etc/slurm/
```
```bash
mkdir /etc/slurm-llnl
```
```bash
cp /tmp/slurm.conf /etc/slurm-llnl/
```
```bash
cp /tmp/slurm.conf slurm-21.08.8/etc/
```
```bash 
systemctl stop ufw
```

```bash
iptables -F
```

```bash
cp slurmd.service /etc/systemd/system
```
```bash
scp -r slurm-21.08.8/etc/cgroup.conf compute@compute:/tmp
```
```bash
cp /tmp/cgroup.conf .
```
```bash
service slurmd start
```
```bash
service slurmd status
```
# in slurm.cof change cgroup Proctype=linuxproc
