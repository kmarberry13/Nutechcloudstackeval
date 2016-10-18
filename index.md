---
title: Index
---

# Table Of Contents
1. [Introduction](#user-content-introduction)
2. [Resources](#user-content-resources)
3. [Cloud Environment](#user-content-choosing-the-cloud-environment)
4. [Hypervisor](#user-content-choosing-the-hypervisor)
5. [Conclusion](#user-content-conclusion)
6. [Building the Cloud](#user-content-building-the-cloud-environment)
    1. [Building from Source](#user-content-building-from-source)
    2. [Setting up the Repository](#user-content-setting-up-the-repository)
    3. [Installing the Management Server](#user-content-installing-the-management-server)
    4. [Installing the Hypervisor](#user-content-installing-the-kvm-hypervisor)
    5. [Setting up the Management Server](#user-content-Setting-up-the-management-server)


## Introduction

This is an evaluation using opensource cloud environment in a small or home office situation. We wanted to evaluated them with consideration of resources available to small or home offices. We also wanted to see if there was possible interoperability with vmware workstation so we could move virtuals between a production and test environment as easy as possible. This also provides some level of redundancy if the cloud environment were to fail. First we evaluated our options between Openstack and cloudstack. We needed to find out what the requirements were and how they fit into what we had available.

[Back to the top](#user-content-table-of-contents)

## Resources

Here is what we had.

1. We had a FreeNAS 9.10 server with 200G of zfs1 storage that was preconfigured for this evaluation with NFS shares. (it is beyond the scope of this paper on the setup of this device)
2. Three servers with:
      1. Intel(R) Xeon(R) CPU: E5520  @ 2.27GHz 4 core with hyper threading turned on for a total of 8 cores
      2. 8Gs of ram each (that is pretty low for production)
      3. Local 1T sata harddrive
3. 100Mbit dumb switch (we did not have a gig switch available, for production would require a gig switch)

[Back to the top](#user-content-table-of-contents)

## Choosing the cloud environment

**Environment:** Openstack vs Cloudstack

||Openstack|Cloudstack|
|:---:|:---:|:---:|
|Equipment required| 3-5 minimum machines | 2 minimum machines |
|Labor required| 1 person per module | 1 person |

Openstack has a very complex modular system, Openstack even states that to really learn and take advantage of Openstack's abilities you need to have at least one person dedicated to learning a single module. Due to this complexity the equipment requirements a higher then Cloudstack.

[Back to the top](#user-content-table-of-contents)

## Choosing the Hypervisor

**Hypervisor** : KVM vs XEN

||KVM|XEN|
|:---:|:---:|:---:|
|Ease of installation| quick, just a few lines of code | heavy, need to follow a long installation guide|
|Equipment required| 2 machines for the hosts | minimum of 3 machines for hosts in case of hosting failures|

[Back to the top](#user-content-table-of-contents)

## Assessment

We came to the decision that with the resources we had available we would only be able evaluate Cloudstack.
With only one person working on this project and only three machines and one NAS it was not feasible to try a work with Openstack.

Once that was decided we looked at the hypervisor. Again looking at what we had to work with we went with KVM. The ease of installation pared with community backing and ever expanding features, it was the choice for this project.

This does not mean that with more manpower or equipment the other options would not be viable. It is just that with what we had, we felt that Cloudstack and KVM were the better choices for the project.

[Back to the top](#user-content-table-of-contents)

# Building the Cloud Environment

There are two ways to build a cloudstack environment. You can choose to use their repository which comes with the generic options. You can also build from source which gives you a greater flexibility of what options you want to add. **We needed to install some non-oss dependencies so that we had added functionality such as Vmware support.** Building using the source allowed us to use non-oss options.

## Building from Source

###### NOTE:  _The recipe below is accurate as of the time of this writing. you may need to get fresh downloads from the appropriate sites if they have changed versions._

#### The Recipe

* The first thing you should do is set up your initial build environment. This should be on one of your bare metal servers.

* You want to have your server with a fresh install of your _Ubuntu 14.04_ Operating System. (this needs to be a brand new OS install onto your bare metal server) **We realized we could not install _Ubuntu 16_ as Cloudstack is just not ready for that version. We had issues with the build code not working with _SQL 5.7_ which comes with _Ubuntu 16_. Cloudstack as of the time of this writing had not updated the SQL syntax that had changed from _SQL 5.5 to 5.7_. so we had to build with _Ubuntu 14.04_, as this is a release that is supported for another three years we decided this was acceptable.**

* So your first step after completing the fresh OS install is to update the apt-get repository.

```
  apt-get update
```

* You will then want to download Cloudstack via their website,

```
wget http://apache.claz.org/cloudstack/releases/4.8.0.1/apache-cloudstack-4.8.0.1-src.tar.bz2
```

* and the KEYS and asc file for authenticating the download.

```
wget http://www.apache.org/dist/cloudstack/KEYS
```

```
wget http://www.apache.org/dist/cloudstack/releases/4.8.0.1/apache-cloudstack-4.8.0.1-src.tar.bz2.asc
```

* Next you will want to verify the downloaded file by doing the following:

```
gpg --import KEYS
```

```
gpg --verify apache-cloudstack-4.8.0-src.tar.bz2.asc
```

* If the output has at least one _**"good Signature"**_ you can continue on. just a reminder the links I gave above are as of this posting. You might want to go to Apache Cloudstack's website and get the current links to run your build.

* So after you have verified that the download is good you need to extract the file.

```
tar -jxvf apache-cloudstack-4.8.0-src.tar.bz2
```

* Once the extraction is complete you need to move into the folder that was created.

```
cd apache_cloudstack_4.8.0.1-src
```

* Next step is installing the dependencies. now this takes some apt-get updates in between installs or some of them can not be found. so do the following updates and installs:

```
   apt-get update
   apt-get install python-software-properties
   apt-get update

   apt-get install  ant debhelper openjdk-7-jdk tomcat6 libws-commons-util-java genisoimage
 python-mysqldb libcommons-codec-java libcommons-httpclient-java
 liblog4j1.2-java maven

   apt-get update
   apt-get install python-setuptools mkisofs mysql-server
   apt-get update
   
 ```

* Then you want to move into the apache cloudstack deps folder.

```
cd deps

```

* The next steps are for building dependencies that are not stored on Cloudstack's website.
You have to go get them from third party devs.

```
wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-iControl.jar
```

```
wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-manageontap.jar
```

```
wget http://zooi.widodh.nl/cloudstack/build-dep/vmware-vim.jar
```

```
wget http://zooi.widodh.nl/cloudstack/build-dep/vmware-apputils.jar
```

```
wget http://zooi.widodh.nl/cloudstack/build-dep/cloud-netscaler-jars.zip
```

```
wget https://raw.githubusercontent.com/rhtyd/cloudstack-nonoss/aa380fd706d4483367c18e2231c4caa94b6ed7b8/vim25_55.jar
```

* Next you want to rename some of those file as the files were named slightly different then cloudstack will be looking for.

```
mv cloud-manageontap.jar manageontap.jar
```

```
mv vmware-apputils.jar apputils.jar
```

```
mv vmware-vim.jar vim.jar
```

* Then you need to unzip the netscaler files

```
unzip cloud-netscaler-jars.zip
```

* After that you need to build those into your maven dependency repository

```
./install-non-oss.sh
```

* Now you need to back up a folder and then run the build dependences command

```
cd ..
```

```
  mvn -P deps # -D noredist
```

* The **-D noredist** is what tells maven to build the dependences with the non-oss dependences as well.
Next you want to build Cloudstack. You can do this with no auto test ran or you can run it with auto tests on. Running it with auto-tests on doubles the build time.

**For building without auto-test run:**

```
  mvn -Dmaven.test.skip=true clean install -e -P systemvm,developer -Dnoredist
```

**For building it with auto-tests run:**

```
  mvn clean install -e -P systemvm,developer -Dnoredist
```

* running the build with or without auto-tests does not change cloudstacks features. It just changes whether or not the build runs checks on the build.

[Back to the top](#user-content-table-of-contents)

## Setting up the repository

**Now that the build is done and baring build failures, you are ready to add them to your repository.**

**Your repository should be set up on the same server you built by source**

* First thing you want to do is install apache 2.0.

```
apt-get install apache2
```

* Next you need to build the cloudstack packages

```
dpkg-buildpackage -uc -us
```

* Then you need dpkg dev, _you might already have it but it does not hurt to try to install it just in case._

```
apt-get install dpkg-dev
```

* You need to make a directory that apache 2.0 can find you cloudstack dependences in.

```
mkdir -p /var/www/html/cloudstack/repo/binary
```

* you next need to copy the following files:

```
cp *.deb /var/www/html/cloudstack/repo/binary
```

```
cp *.changes /var/www/html/cloudstack/repo/binary
```

```
cp *.dsc /var/www/html/cloudstack/repo/binary
```

* Move to the repository folder.

```
cd /var/www/html/cloudstack/repo/binary
```

* You now create the packages.

```
sh -c 'dpkg-scanpackages . /dev/null | tee Packages | gzip -9 > Packages.gz'
```

* The next steps you need to make sure yo do to _**every machine that will be building with cloudstack**_.
* Create the file below:

```
vim /etc/apt/sources.list.d/cloudstack.list
```

* Add the following line to cloudstack.list.

```
http://10.1.10.125/cloudstack/repo/binary ./
```

* Now to make sure you have the cloudsack packages you have to run:

```
apt-get update
```

* The following steps are so that the machines can talk to each other. Make sure you add the ip addresses of _**each machine you will be using**_.

```
vim /etc/hosts
```

```
client name      ip address
```

_The machine you are working on will have an example of the line of code as it knows itself. Just follow its example._

[Back to the top](#user-content-table-of-contents)

## Installing the management server

###### Note: _The management server should be a machine that is dedicated to just managing your cloudstack environment, and if desired a usage server. It should be another of your bare metal servers (one not already being used). The management server is **NOT** used for hosting Vms._

* You should start with a brand new install of Ubuntu 14.04.
* The first thing to do is update you apt-get repository.

```
apt-get update
```

* Next you need to install a few dependencies

```
apt-get install screen openssh-server openntpd
```

_screen is used to be able to have others monitor what you are doing and to keep a log of your commands for later review. Openssh-server is so you can ssh into the management server later on, and openntpd will help the management server keep proper time._

_To setup the ssh environment you need to do a few things._
* First is setup your su password.

```
passwd root
```

_This sets the roots password, make sure you use a password you will remember._

* Next you need to go into the ssh config files and change one line.

```
vim /etc/ssh/sshd_config
```

* change _PermitRootLogin_ from **without-password** to **yes**

* Then you need to restart your ssh service

```
service ssh restart
```

**Make sure you setup your repository file on this machine.**

```
vim /etc/apt/sources.list.d/cloudstack.list
```

* add line to cloudstack.list

```
deb http://10.1.10.131/cloudstack/repo/binary ./
```

* update packages so that you will now have access to cloudstack dependences

```
apt-get update
```

* edit hosts file so that it can talk to the other machines

```
vim /etc/hosts
```

```
client name      ip address
```

**Now we are ready to install the managment server.**

```
apt-get install cloudstack-management
```

* Download vhd-util

```
wget  http://download.cloud.com.s3.amazonaws.com/tools/vhd-util
```

* Copy the file over into the cloudstack-common folder.

```
cp vhd-util /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver
```

* Install mysql-server

```
apt-get install mysql-server
```

* Configure the mysql cnf file.

```
vim /etc/mysql/my.cnf
```

**Put these lines below the _datadir_ line**

```
innodb_rollback_on_timeout=1
```

_This sets it so the database rolls back if there is no response in a certain time frame_

```
innodb_lock_wait_timeout=600
```

_This sets the roll back timer to 600 seconds._

```
max_connections=350
```

_This sets the max number of connections. Cloudstack states max of 350 connection per management server._

```
log-bin=mysql-bin
```

_this sets the log-bin file to be mysql-bin_

```
binlog-format = 'ROW'
```

_This sets the events to be ROWs so it looks at the rows to determine events._

* restart the mysql-server.

```
service mysql restart
```

* setup database.
_You need to replace the values inside the **<>** with something or nothing **(remove the <> as well)** if you choose to put nothing in then the password will be blank for both the root and the database._

```
cloudstack-setup-databases cloud:<dbpassword>@localhost --deploy-as=root:<password>
```

* Now we can start the cloudstack management server.
**NOTE:** _this is **NOT** the setup. This is just the **install** and startup. The setup will be discussed later in the document._

```
cloudstack-setup-management
```

* Next you need to set up you NSF share to be able to accept the cloudstack generic templates. without them you can not start a cloudstack instance.
* First on the management server you need to make a mnt directory for you secondary storage share.

```
mkdir -p /mnt/secondary
```

* Next you need to attach it to your NSF shared device. (our was our preconfigured NAS)

```
mount -t nfs nfsservername:/nfs/share/secondary /mnt/secondary
```

* Now we need to install the hypervisor templates.
* If you are only going to use a single hypervisor you only need to install that one template.


* For Hyper-V
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /mnt/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.6/systemvm64template-4.6.0-hyperv.vhd.zip \
-h hyperv \
-s <optional-management-server-secret-key> \
-F
```
* For XenServer:
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /mnt/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.6/systemvm64template-4.6.0-xen.vhd.bz2 \
-h xenserver \
-s <optional-management-server-secret-key> \
-F
```
* For vSphere:
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /mnt/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.6/systemvm64template-4.6.0-vmware.ova \
-h vmware \
-s <optional-management-server-secret-key> \
-F
```
* For KVM:
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /mnt/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.6/systemvm64template-4.6.0-kvm.qcow2.bz2 \
-h kvm \
-s <optional-management-server-secret-key> \
-F
```
* Once you have all the template for the hypervisors you require. you need to unmount then remove the secondary mnt directory.
```
umount /mnt/secondary
rmdir /mnt/secondary
```

* Now you are ready to install a hypervisor and then set up the management server.

[Back to the top](#user-content-table-of-contents)

## Installing the KVM Hypervisor
**This needs to be installed on a unused bare metal server. This will be used to host your VMs.**

**You should start with a brand new install of Ubuntu 14.04.**
* The first thing to do is update you apt-get repository.

```
apt-get update
```

* Next you need to install a few dependencies

```
apt-get install screen openssh-server openntpd
```

_screen is used to be able to have others monitor what you are doing and to keep a log of your commands for later review. Openssh-server is so you can ssh into the management server later on, and openntpd will help the management server keep proper time._

_To setup the ssh environment you need to do a few things._
* First is set up your su password.

```
passwd root
```

_This sets the roots password, make sure you use a password you will remember._
* Next you need to go into the ssh config files and change one line.

```
vim /etc/ssh/sshd_config
```

* Change _PermitRootLogin_ from **without-password** to **yes**

* Then you need to restart your ssh service

```
service ssh restart
```

**Make sure you setup your repository file on this machine.**

```
vim /etc/apt/sources.list.d/cloudstack.list
```

* Add line to cloudstack.list

```
deb http://10.1.10.131/cloudstack/repo/binary ./
```

* Update packages so that you will now have access to cloudstack dependences

```
apt-get update
```

* Edit hosts file so that it can talk to the other machines

```
vim /etc/hosts
```

```
client name      ip address
```

**Now it is time to install the hypervisor.**

```
apt-get install cloudstack-agent
```

**You need to configure a few things inside libvirtd.conf**

```
vim /etc/libvirt/libvirtd.conf
```

* Set the following parameters

```
listen_tls = 0
listen_tcp = 1
tcp_port = "16509"
auth_tcp = "none"
mdns_adv = 0
```

* Next you need to change one line in libvirt-bin.

```
vim /etc/default/libvirt-bin
```

* Set the following parameters.

```
libvirtd_opts="-d -l"
```

* Restart libvirt

```
service libvirt-bin restart
```

**Now we can start our cloudstack agent**

```
service cloudstack-agent start
```

**We are now ready to setup the management server.**

[Back to the top](#user-content-table-of-contents)

## Setting up the management server

## NOTE: Make sure you have at least one machine set up with a hypervisor, for us it was KVM. Also make sure you know the range for your private network and public networks. _Once set they are hard to change._

**log into the management server**

_In a web browser :_

_"managementserver ipaddress":8080/client_

**First time will be user name:  _admin_     pw: _password_**

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-00-27.png?raw=true)

**You can choose to either skip the guide or continue with basic installation. We chose to continue with basic installation.**

* It asks you to change the password.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-07-07.png?raw=true)

* Lets first set up a zone.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-07-46.png?raw=true)

* Asks for a name for the zone, and 2 dns addresses
    * a dns server for the public guests so a public dns
    * a dns server for the private vms so a private dns

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-08-17.png?raw=true)

* Next we setup a pod.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-30-44.png?raw=true)

* It now asks for a private network range.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-20%2010-06-36.png?raw=true)

* It asks for the public network range.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-20%2010-07-29.png?raw=true)

* Next we setup a cluster.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-34-22.png?raw=true)

* Pick a hypervisor and a name.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-35-10.png?raw=true)

* Next we setup one host.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-35-31.png?raw=true)

* Put in the host you already setups ip address and username and password.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-36-04.png?raw=true)

* Next we setup the primary storage.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-37-25.png?raw=true)

* Put in your storage name zone-wide or cluster, then the ip address of your storage, then your mnt folder you setup.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-39-18.png?raw=true)

* next we setup the secondary storage.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-39-45.png?raw=true)

* put in the ip address of the storage and the path to the mnt folder.

![](https://github.com/kmarberry13/sohocloudstackeval/blob/gh-pages/images/Screenshot%20from%202016-07-19%2014-40-28.png?raw=true)

**Then you click on the launch button and it will build your zone with all its parts.**

[Back to the top](#user-content-table-of-contents)
