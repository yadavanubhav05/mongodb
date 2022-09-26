/!\ Please note that this playbook will be updated on timely manner. /!\

Role Name
=========

This playbook deploys MongoDB with replica Sets (with replica name initialized)
with SSL : self signed certificates
OR
Without SSL (specify in Variables)
Requirements
------------

Refer MongoDB docs https://docs.mongodb.com/v4.4/administration/production-notes/

Role Variables
--------------
#Pre-requisites
replSetName
baseurl  (download rpms https://repo.mongodb.org/yum/redhat/7/mongodb-org/ )
gpgkey  ( for future releases, download gpgkey from https://www.mongodb.org/static/pgp/ )
DOMAIN

#Mongo specific
db_path
mongo_ssl (true or false)

#SSL specific
country_name
state
city
org_name
ou_name
email
challenge_password

Dependencies
------------

Not yet

Example Playbook
----------------
Before running the playbook, please ensure:
/tmp/mongo  is created on your controller host.
Most important --> DNS works very well across all nodes. (otherwise you will notice SSL handshake errors)

For non-SSL requirements, run as below and remember to set mongo_ssl: "false" in Variables.
ansible-playbook -i inventory deploymongo-ssl.yml --skip-tags=ssl,ca

For SSL requirements, run as below and remember to set mongo_ssl: "true" in Variables.
ansible-playbook -i inventory deploymongo-ssl.yml

After running this playbook;
remove /tmp/<hostname_initials> from the controller node.

sample inventory
----------------
[rootca]
hostname ansible_host=IP  #this host needs to be strictly different from hosts in mongodb
[mongodb]
hostname ansible_host=IP
[controlmachine]
hostname ansible_connection=local  #any VM from rootca or mongodb

Ref above inventory file - please note that MongoDB will be installed on mongodb + controlmachine groups. (except rootca node, rootca will be an independent node)

Hostnames need to be short names (without Domain name)

A dummy inventory looks like below

[rootca]
mongodb4-ves001 ansible_host=IP
[mongodb]
mongodb4-vhm001 ansible_host=IP
mongodb4-vhm002 ansible_host=IP
mongodb4-vhm003 ansible_host=IP
[controlmachine]
controlmachine ansible_connection=local

Freebies ;)
-------
on all nodes, run
mongo;

rs.initiate()

use admin;
db.createUser({user:"mongoadmin",pwd:"password",roles:["clusterAdmin","readWriteAnyDatabase","dbAdminAnyDatabase","userAdminAnyDatabase"]})

use admin;
switched to db admin
mongors01:PRIMARY> db.createUser({user: "mongoadmin", pwd: "password", roles:[{role: "root", db: "admin"}]})
Successfully added user: {
        "user" : "mongoadmin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}

to connect to mongo shell:
without ssl
mongo;

with ssl
mongo --ssl --sslCAFile /etc/mongo/ssl/MongoCA.crt --sslPEMKeyFile /etc/mongo/ssl/ayadav-ves001.pem --host ayadav-ves001.data.ayadav

I had to struggle a lot with below:

Before connecting to SSL, disable authorization (rset.key)
comment this block and restart service.

to completely remove MongoDB and leftovers, beware this is highly destructive and there is no reversal.

ansible -i /tmp/mongo1 all -m shell -a "yum remove mongo* -y ; rm -rf /var/log/mongodb/*;rm -rf /var/lib/mongo/*;rm -rf /etc/mongo/ssl/*; rm -rf /tmp/mongodb-27017.sock; rm -rf /etc/mongo/rset.key; rm -rf /etc/mongo/ssl/*" -b

BSD

Author Information
------------------
Anubhav YADAV
Contact yadav.anubhav05@gmail.com 
