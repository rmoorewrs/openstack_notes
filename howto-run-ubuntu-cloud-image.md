# How to Use an Ubuntu Cloud Image in Titanium/OpenStack
Author: richard.moore@windriver.com

Cloud images are much easier to use than installing from scratch and are typically maintained by the Linux distribution publisher, like Canonical for Ubuntu.

This tutorial uses remote clients (delivered through a docker container) but most of the operations can also be done through Horizon.

### 1 ) Download bionic cloud image from [Ubuntu Cloud Repo](https://cloud-images.ubuntu.com/bionic/current/)
https://cloud-images.ubuntu.com/bionic/current/
```
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```
### 2 ) Run the tic-sdk container with the remote clients (skip this if you already have the remote clients from the TiC SDK installed).
>Note: your host must have docker installed. This works on Windows 7,10/Mac/Linux if docker is installed correctly.

#### 2.1) Download the OpenStack Credentials From Horizon
Download the OpenStack credentials file by logging into Horizon with the project and user you want to use. 
```
Project->Compute->Access & Security->API Access->Download OpenStack RC File v3
```
Place the cloud image and credentials in the same directory, where you will run the TiC remote client shell.

#### 2.2) Run the TiC Remote Client Shell in Docker 
> Note:  You can skip running the TiC remote client shell in docker if you already have the TiC remote clients installed natively from the TiC SDK on WindShare. Native installation is only supported on Ubuntu and RedHat/CentOS.

- Open a shell and cd into the working directory containing the credentials files, images, etc that you want use. 
- Run the remote client shell in docker. 
>The first time will take some time as the image downloads. Subsequent runs will start very quickly since the image in local cache will be used. 

In a bash shell: (Linux, Mac, Windows cygwin/git bash shell/etc)
```
docker container run --rm -it -v $(pwd):/home/wruser/host rmoorewrs/tic-sdk
```
In Windows CMD Shell:
```
docker container run --rm -it -v %cd%:/home/wruser/host rmoorewrs/tic-sdk
```
### 3 ) Source the OpenStack Credentials File
Whether you're running the tic-sdk docker container or using the TiC remote clients installed natively, you need to source the openrc credentials file for your user/project:
- substitute the name of your file for `my_openrc.sh`
```
source rc/my_openrc.sh 
Please ... or press enter if you are not using HTTPS: <enter>
Please enter your OpenStack Password for project admin as user admin: <password> <enter>
```
>Note: it is possible to have multiple OpenStack credentials files (rc file), one for each server/project/user. However, only one set of credentials can be active at a time within the shell. In order to work with a different server/project/user, simply source a different rc file.

### 4 ) Upload the cloud image file with `glance image-create` like this:
```
glance image-create --visibility public --container-format bare --disk-format qcow2 --progress --file bionic-server-cloudimg-amd64.img --name bionic-server-cloudimg-amd64.img
```
When finished, you can check the availability of the image with:
```
glance image-list
...
| 96eae3aa-9598-4b9c-9e3d-6eb5c3eb3e1a | bionic-server-cloudimg-amd64.img       |
...
```
### 5) Create a keypair to use with the cloud image 
>Note: skip this step if you have an existing keypair and have access to the private key of the keypair.
```
openstack keypair create ubuntu_bionic_keypair
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA0KVB6w7b6Zf9821fbPoJ1sE6HYOWGaD0eJ30TqBiB2w2dzpL
+g9tFyz5/W/2RgqRGLxN266Am0RhW8BbkKutFce3KXE5HFaXmPv58FpKDW3yxRCS
...
KGG6jmEtQk25mD8tFnKXuocez1hIzgZTVuGF4wIEsqkqC5LSqnmC9nA23ZVTEYsF
7hhSQqclY9OV/bETTQKzmONhrLVo025Qpf5iicOGCMtHdageSk8D
-----END RSA PRIVATE KEY-----
```
#### 5.1) Copy and paste the private key shown on the screen into a text editor and save 
 ***NOTE: This is your only chance to save the private key!*** 
- Include the lines with the `---`
- Set the security of the private key (assume you called it `ubuntu_bionic_private.pem`)
```
chmod 600 ubuntu_bionic_private.pem
```
>Note: if you skip this step, ssh will not accept the private key file.

### 6) Launch the Instance from the Ubuntu Bionic image using  your favorite method (horizon, heat, nova, etc)
- make sure to specify the keypair you created in step 5 (or the keypair you intend to use) when launching the instance
- make sure the instance is on a network that's routable from your current machine (use floating IP if necessary)
- find the externally available IP address of the instance
>NOTE: you won't be able to log in via the console or by  ssh without using the private key

#### 6.1) Connect to the Instance using ssh and the private key
```
ssh -i ubuntu_bionic_private.pem ubuntu@<instance IP addr>
```
You should now be able to log in with no other password

### 7) Configure the Ubuntu Instance to allow password logins
See full instructions [here](https://fosskb.in/2015/02/06/enable-password-authentication-on-cloud-images/):
```
https://fosskb.in/2015/02/06/enable-password-authentication-on-cloud-images/
```
#### 7.1) Edit /etc/ssh/sshd_config to allow Password Authentication:
```
sudo vi /etc/ssh/sshd_config
```
Change `PasswordAuthentication` to yes:
```
PasswordAuthentication yes
```
#### 7.2) Set a password for `ubuntu` and/or create a new user
By default the `ubuntu` user doesn't have a password, so set it:
```
sudo passwd ubuntu
```
**and/or** create a new user (make sure and set the password):
```
sudo useradd -m -s $(which bash) -G sudo mynewuser
sudo passwd mynewuser
```
#### 7.2) Restart sshd
```
sudo systemctl restart sshd
```
#### 7.3 Logout and Log back in with new user (or ubuntu)
If all went well, you should now be able to log in without the private key
```
ssh ubuntu@<instance_ip_address>
```

That's it!

