# Setting up the postgres and running hammerdb benchmark using ansible

## Introduction
In this tutorial we are setting up the psotgres and after that running hammerdb benchmark on the openshift virtual machine. 
* We are using ansible to automate all the steps.
* I have tested the steps documented by running ansible from my personal laptop. i.e., macbook pro m3.

### Set up the openshift VM with repos and ansible
Once you have the VM from openshift virtualization platform [register](https://console.redhat.com/insights/connector/activation-keys) with redhat to configure the repo.
[//]: # (adding the rhel repos to install anything.)
```shell
sudo subscription-manager register --activationkey=<> --org=<>
```

You need to install the python3 so that ansible automation can work. If you are using rhel8 then may be you need to upgrade python version.
```shell
sudo yum install -y python3
```
find the path of the python3 and add it to ansible inventory
```shell
which python3
```

If you can't expose the VM externally but need to run Ansible, you can port-forward SSH from the VM to your local machine:

```shell
virtctl port-forward <vm-name> 2222:22 -n <namespace>
```

you can also SSH from your local machine as below:
```shell
ssh rhel@localhost -p 2222
```

Now add below VM to ansible inventory.ini. `ansible_ssh_private_key_file` is the private key to do the ssh in to VM. This path needs to be corrected as per your scenario.
```shell
openshift-vm ansible_host=127.0.0.1 ansible_user=rhel ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_port=2222 ansible_python_interpreter=/usr/bin/python3
```

Check if the ansible able to reach the VM by doing below simple test
```shell
ansible -i inventory.ini -m ping openshift-vm
```

If above statement works fine then your ansible setup is successful. And you are good to configure postgres and hammerdb.

### Setting up the postgres on the VM

Run below playbook to set up the postgres
```shell
ansible-playbook -i inventory.ini setup_postgres.yml
```

### Setting up the hammerDb on the vm and running a benchmark

Run below playbook to set up the hammerdb and run a benchmark on postgres

```shell
ansible-playbook -i inventory.ini install_setup_hammer_db.yml
```

## Debugging hammerdb issues
Refresh hammerdb cache if the test does not reflect your configuration changes.
```shell
#refresh hammerdb cache/configs
rm /tmp/database.db
```

Often hammerdb loads the default configurations so it is useful to see the effective configuration using below commands.
```shell
#run in interactive mode
./hammerdbcli

#enter below command to see all the effective configurations.
print dict
```

## Some helpful postgres commands

```shell
# connect to the postgres database on the vm
PGPASSWORD='mypassword' psql -h localhost -p 5432 -U postgres -d postgres

# connect to the tpcc database on the vm
PGPASSWORD='mypassword' psql -h localhost -p 5432 -U postgres -d tpcc

# drop and recreate the tpcc database after each run.
DROP DATABASE IF EXISTS tpcc;
CREATE DATABASE tpcc;

#displays all the tables in current database
select * from information_schema.tables

#restarting the postgres service
sudo systemctl restart postgresql
```