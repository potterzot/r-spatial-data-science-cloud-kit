## Setting up a spatial data science instance on AWS

There are two AWS services in particular that are necessary for setting up a spatial data science virtual machine: __EC2__ and __S3__. 
EC2 handles the actual virtual machines, S3 handles data storage.

### Step 1: Setting up AWS infrastructure, the least fun part

#### First you need a root account, a group, and a user account

Create a root account here: [https://aws.amazon.com/](https://aws.amazon.com/). 

Once you've done that, the services dropdown menu at the AWS Console Interface](https://console.aws.amazon.co) provides access to [Identity Access and Management (IAM)](https://console.aws.amazon.com/iam) as well as EC2 and S3 services.

You need acess policies, a group, and a standard user account. All of which are added via [IAM](https://console.aws.amazon.com/iam). 

__Group Account__:

1. Create the group name (e.g. "users")
2. Attach two policies: EC2-FullAccess and S3-RWAccess. These allow group members to use any EC2 resources and to read and write to S3 "buckets". There are many more options if finer control is needed.

__User Account__:

1. Create a user account with programmatic access (allows CLI), assign them to the "user" group, and download the resulting security key file.
2. Download the authentication key and store it somewhere safe. It should be a file that has the "pem" file extension.

__IAM Role__:

1. An IAM role makes it easy to assign a group of permissions to a virtual machine, so create that and add both

#### Set up S3 buckets for data storage

1. You can create multiple buckets that can be managed differently (e.g. whether there is public access), but for initial use a bucket with the default settings is fine. The bucket will be where data is stored when not used as part of a EC2 virtual machine.


#### Set up an EC2 instance that you will customize by adding additional software

1. In the EC2 interface, click on Instances and "Launch Instance". This guide uses the Ubuntu instance, but there are others optimized for Deep Learning (see [sagemaker](https://aws.amazon.com/sagemaker) for more, but IMO it's too complex for standard research needs).
2. Choose the free micro tier, since we'll just be adding software and getting things set up at this point.
3. Choose configuration options and select a VPC and IAM Role. Both can be created with the default settings.
4. In the next "Add Storage" step, create a new hard drive with the default settings. If you want encryption you can enable that here.
5. Skip "Add Tags" and go to "Configure Security Group". Create a new security rule that allows: (1) SSH from your IP and (2) HTTPS from your IP. You can change these later.
6. You can create a new key pair or select an existing one. This will allow you to connect to your instance via the terminal.

You should be redirected to an instance page. Click on "Connect" and "SSH Client" which will provide the command to connect via terminal. It looks like:

`ssh -i "<key file>.pem" ubuntu@ec2-34-223-229-211.us-west-2.compute.amazonaws.com`


### Step 2: Creating your spatial data science image

Now the fun part! We begin by updating packages and then install the relevant software: emacs, R, RStudio, RStudio Server, python and jupyter, spatial processing packages, etc...

1. Update packages: `sudo apt update; sudo apt upgrade`
2. Install emacs: `sudo apt install emacs-nox`
     sudo apt install elpa-ess ess
     upload .emacs.d/init.el
     install: smartparens, smart-mode-line, solarized-theme, smart-mode-line-atom
3. Install geo packages: `sudo apt install gdal-bin libgdal-dev libgeos-dev libproj-dev`
4. Install other packages: `sudo apt install libudunits2-dev procinfo`
5. Install AWS CLI: `sudo apt install awscli`
    aws configure # the info for this comes from: https://console.aws.amazon.com/iam/home
6. Prep for installing R:
    sudo apt install apt-transport-https software-properties-common
	sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
	sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
	sudo apt update
	sudo apt install r-base
7. Install R packages that you want on your base install: `sudo -i R`
    install.packages(c("tidyverse", "devtools", "data.table", "sf", "geosphere", "rgdal", "proj4", "rgeos", "here", "tigris", "maps"))
    install.packages(c("spdep", "nmle", "plm", "lmtest", "MuMIn", "spatialreg", "fixest", "lfe")
8. Install Python libraries:
    sudo apt install python3-pip python3-dev
	sudo -H pip3 install --upgrade pip
	sudo -H pip3 install virtualenv
2. scp config files from your local machine:
    .emacs.d/init.el
    .gitignore .gitconfig
    .Renviron
    .R/Makevars
3. Import gpg key for github:
    gpg --list-secret-keys <your email>
    gpg --export-secret-keys <KEY ID> > private.key
    scp 
    gpg --import private.key
    export GPG_TTY=$(tty) and/or add to .bashrc

### More that needs to be cleaned up

## Appendix III: Attaching and mounting an EBS volume

We have a separate volume for data. The volume is attached via the aws console
1. Attach the volume via aws: ``

If it's a new volume, then format and mount:
2. fdisk /dev/xvdf -> new partition
3. mkfs.ext4 /dev/xvdf1
4. sudo mount /dev/xvdf1 /data
5. sudo chown ubuntu /data

#### Jupypter Notebook / JupyterLab

```
   # Install jupyter notebook within a project
   virtualenv <my project>
   source <my project>/bin/activate
   pip install jupyter
   jupyter notebook --generate-config
   jupyter notebook password
   
   # Generate a server certificate
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mykey.key -out mycert.pem
   jupyter notebook --certfile=mycert.pem --keyfile mykey.key
   
   # See here for additional steps
   #https://jupyter-notebook.readthedocs.io/en/stable/public_server.html
   # In particular:
   # - edit ~/.jupyter/jupyter_notebook_config.py to add cert and key
   # - edit ~/.jupyter/jupyter_notebook_config.py to set c.NotebookApp.open_browser = False
   
   # From local machine, connects and forwards localhost:8888 from the server to 8889 on local:
   ssh -i ~/.ssh/<aws key> -L 8889:localhost:8888 <ec2 user>@<ec2 ip4 addr>
   
   # Now in same terminal:
   source <myproj>/bin/activate
   jupyter notebook
```









	
	
