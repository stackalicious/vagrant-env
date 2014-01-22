Stackalicious Single-Node Vagrant LAMP Environment
=============

This project simplifies the process of web development in a couple ways:
* The server environment is defined by a base image overlaid with an install script so every deployment contains exactly what you need and nothing else.
* Since the server environment is virtual we can deploy to different targets.  Specifically a local virtual machine for testing and a remote/global deploy to HP Public Cloud.
* Our install scripts are split into two components: platform/environment setup and app specific setup.  This allows for developers to promote and share best practices at the platform level while keeping their application details private.

This stack script installs the following components:
* Ubuntu Linux (12.04)
* Apache2 (2.2.22)
* MySQL (5.5.34)
* Php5 (5.4.9)

Prerequisites
------------------
This projects assumes you have already installed the following:
* git (http://git-scm.com/downloads) (ex: v1.8.4.3)
* VirtualBox (https://www.virtualbox.org/wiki/Downloads) (ex: v4.3.2)
* vagrant (http://downloads.vagrantup.com/) (ex: v1.4.3)

Note: On Windows when installing Git it is recommended that you select the option to include the bin directory in your path to use the vagrant ssh command.  Otherwise you can use a differt ssh client such as PuTTY.

Install
-----------------
Now add a few plugins to your vagrant environment from the commandline:
* `vagrant plugin install vagrant-vbguest` Ensures that the "guest additions" on the guest os is up to date. 
* `vagrant plugin install vagrant-omnibus` Installs ruby and chef on the guest os allowing the vagrant script to complete.
* `vagrant plugin install vagrant-hp` (optional) Adds ability to deploy to HP Public Cloud


Stackalicious Vagrant environment contains a 'sites' directory where you can provide your own website files to be hosted in the virtual environment.  As an example the following instructions will install a simple website called campapp.

> cd vagrant-lamp/sites  
> git clone git@github.com:dnielsen/campapp.git  
> cd .. 
> vagrant up local

Once complete you will have a complete server environment hosting your website from within VirtualBox.  You will be able to view the website only from the current machine by pointing your browser to 192.168.56.3.

Any changes you make to the web app will be automatically updated within the virtual machine and reflected in the browser immediatly.

Deploy to Cloud
---------------
Stackalicious script also provides support for cloud deployment.  Today we support only HP Public Cloud deployment.  To get started you need to have a HP Cloud (http://www.hpcloud.com/) account set up.

Next you will need to provide the following information from your HP Cloud account to the Vagrantfile:
* access_key  = (key id),  In the dropdown at top right by 'Horizon Preview' select 'Manage Access Keys' use the key id here.
* secret_key = (secret key),  Select 'Show Secret Keys' from page above and copy in the secret key.
* flavor   = eg "standard.small"  See the 'Launch Instance' panel (here)[https://horizon.hpcloud.com/project/instances/] for details.
* tenant_id = (project id), Use the id listed on [this page](https://horizon.hpcloud.com/control_services/projects/).
* server_name = eg "vagrant-test", machine name for the new server instance  
* image    = eg "Ubuntu Precise 12.04 LTS Server 64-bit 20121026 (b)", predefined vm image provided by HP  
* keypair_name = eg "hp-vagrant-keypair", Create from the key pairs tab of [Access & Security](https://horizon.hpcloud.com/project/access_and_security/) page.  
* ssh_private_key_path = eg "~/.ssh/hp-vagrant-keypair.pem", File path to key, generated by above step
* ssh_username = "ubuntu", user name from HP vm images
* availability_zone = "us-west" or "us-east", More options are coming in the future.

Set these values in the :hp provider block of the Vagrantfile, towards the bottom of the file.

Once setup you can deploy using the command 'vagrant up remote --provider=hp'.  Once complete run 'vagrant ssh-config remote' to retrieve the external ip address, listed as the 'HostName'.  This ip address is what you enter into the web browser.  To access the deployed server run 'vagrant ssh remote' similar to the local deployment scenerio.



Troubleshooting
------------------
Let's face it web server environments are complicated.  There are many moving parts and sometimes things go wrong. In order to deal with these situation we describe some common stratigies below.

Sometimes the web app files don't synchronize between the host and guest operating system. Rerunning the deployment script can resolve this, 'vagrant provision local'.  

To directly access the server environment use 'vagrant ssh local' or if all else fails you can delete the entire virtual machine with 'vagrant destroy local'.

All of the above commands can be used in the HP Cloud environment by replacing the 'local' parameter with 'remote'.



Q: How do I see the webpage  
A: Open your web browser and type '192.168.56.3' for the url.  This is the static ip assigned to the vm from Vagrantfile.  

Q: Can I directly access the virtual machine?  
A: Graphical access is not enabled. To access from the command line type `vagrant ssh (local|remote)` from the vagrant-lamp directory. For Windows, if ssh is not available from the command line, you will need to use a 3rd party ssh client such as PuTTY. Connect using the static ip 192.168.56.3 port 22 and login with username vagrant, password vagrant.  

Q: How is the server configured?  
A: The Vagrantfile script defines memory and operating system parameters.  The `setup_env.sh` is used to install and configure primary applications: apache, php and mysql.  Finally sites are expected to have a `setup_app.sh` which configures the apache virtual host and populates the database in the  campapp example.

Q: How can I make changes to the site?  
A: You can change the website directly from the sites folder and refresh the browser to see the results.

Q: How do I shutdown/restart/startover the virtual machine in case things go wrong?  
A: `vagrant halt (local|remote))` / `vagrant reload (local|remote)` / `vagrant destroy (local|remote)`  Also if you want to rerun the script use `vagrant provison (local|remote)`.  Not all of these functions are currently supported by the HP provider.

