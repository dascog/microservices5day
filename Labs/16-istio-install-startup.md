# Install and Run Istio on your VM

## VM Setup
- I am using the following
  - AWS Linux Desktop with Docker (ami-03ea45db3152496fc)
  - A t2.large Instance type
  - Allowing SSH, HTTPs and HTTP traffic
  - 30GiB storage.
- Connect using EC2 Instance Connect in the aws console

```
        $ # update everything
        $ sudo yum update
        $
        $ # create a user
        $ useradd trainer
        $ passwd trainer
        $ 
        $ # allow password login (i.e. not requiring public key)
        $ vi /etc/ssh/sshd_config
        $ ## find the "#PasswordAuthentication yes" row and uncomment, and comment out "PasswordAuthentication no".
        $ ## save the file and restart the service
        $ sudo service sshd restart
        $
        $ # add trainer to sudoers
        $ visudo
        $ ## In the file find the line "root ALL=(ALL)  ALL" and add below it "trainer ALL=(ALL)  ALL"
        $ 
        $ # Install docker-compose
        $ sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
        $ sudo chmod +x /usr/local/bin/docker-compose
        $ docker-compose version
        $ 
        $ # Change the docker.sock permissions (for minikube tunnel) and add the users to the docker group
        $ chmod 666 /var/run/docker.sock
        $ usermod -aG docker trainer && newgrp docker
        $ # If you have any other users (e.g. grad):
        $ usermod -aG docker grad 
        $
        $ # Install minikube
        $ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
        $ sudo install minikube-linux-amd64 /usr/local/bin/minikube
        $ minikube version
        $
        $ # log in on VS Code
        $ ## Open the command palette and search for Remote SSH: Add Host
        $ ## Insert "ssh trainer@<your-instance-dns> -A"
        $ ## when prompted select your ~/.ssh/config file to update
        $ ## Search for Remote SSH: Connect to Host and select your host from the list
        $ ## Select Linux as the destination type, and put in your password when prompted.
        $ ## that's is, you're in!
        $
        $ # set alias for kubectl
        $ echo 'alias kubectl="minikube kubectl --"' >> ~/.bashrc
        $ source ~/.bashrc
        $ kubectl version
        $
        $ # download istio
        $ curl -L https://istio.io/downloadIstio | sh -
        $ cd istio-1.14.1
        $ echo "export PATH=$PWD/bin:$PATH" >> ~/.bashrc
        $ source ~/.bashrc
        $ istioctl version
        $
        $ # start minikube cluster
        $ minikube start

## SSH into your VM
- Open a Bash shell on your local machine (if you are on a windows machine and don't have a bash shell, install at https://gitforwindows.org/). Make sure you install OpenSSH as part of your install process.
- Find your VM name on the list and ssh into the machine using ``$ ssh grad@my-machine-name.conygre.com -A``. The password is ``c0nygre`` (second character is a zero).

## set up your installation
- your machine should have minikube installed: ``$ minikube version``
- Perform the following steps to set up an alias for the kubectl command line
```
    $ echo 'alias kubectl="minikube kubectl --"' >> ~/.bashrc
    $ source ~/.bashrc
```
- download and setup istio:
```
    $ curl -L https://istio.io/downloadIstio | sh -
    $ cd istio-1.14.1
    $ echo "export PATH=$PWD/bin:$PATH" >> ~/.bashrc
    $ source ~/.bashrc
    $ istioctl version
```
## Check that Docker is installed
- ``$ docker version``

## Start your cluster
- ``$ minikube start``
- 
## Follow the Istio Tutorial
- Follow the steps at https://istio.io/latest/docs/setup/getting-started/ starting from **Install Istio**
- Note: The URL in the "Verify external access" section will not function due to settings in your EC2, you can verify the application is working via
```
    $ curl http://$GATEWAY_URL/productpage
```

## Cleanup
- Don't worry about this, the instances will be deleted shortly!





