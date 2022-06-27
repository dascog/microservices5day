# Create a Docker Swarm Cluster

## Step 1: Create 3 Docker Instances
- This is best done in the play with docker labs: https://labs.play-with-docker.com/. Other platforms (e.g. AWS VMs) may not have the appropriate security settings to allow traffic between nodes.
- In your play with docker lab, create 3 instances by clicking on ``Add new instance`` in the left hand sidebar.

## Step 2:  Initialize the swarm
- Choose one of your instances (say node 1) and copy the IP address (you can copy and paste in the linux shell of play with docker by selecting text with your mouse and typing CTRL-INSERT to copy and SHIFT-INSERT to paste)
- Type the following command, substituting your node 1 IP address in the correct place:
```
    $ docker swarm init --advertise-addr <NODE-1-IP-ADDRESS>
```

## Step 3: Inspect the current state of the cluster
- Use ``docker info`` to see the current state of the cluster. 
  - There should be no containers running, but ``Swarm:`` should be marked ``active``.
- Use ``docker node ls`` to see the nodes that are currently running in you swarm - you should have only one.

## Step 4: Join the worker nodes to the swarm
- Type ``docker swarm join-token worker`` into your manager shell
- Copy the string printed out and enter it into each of your two worker nodes.
- Use ``docker node ls`` in the manager node to see the whole swarm cluster.

## Step 5: Create a service on the manager node
- We want to create a service that runs the critical function of printing out a ``.`` every second.
- The linux command we need is ``/bin/sh -c "while :; do printf '.'; sleep 1; done"``
- Type the following into your master node to initiate this service:
```
    $ docker service create --replicas 1 --name dotty alpine /bin/sh -c "while :; do printf '.'; sleep 1; done"
```
- You can inspect the service with ``docker service inspect --pretty dotty``

## Step 6: Scale the service
- You can now scale the service to run on other nodes in your cluster:
```
    $ docker service scale dotty=5
```
- You can see the running services using ``docker service ls dotty`` on the manager.
- If you go into the worker nodes and type ``docker ps`` you will also see the containers running on each worker.

## Step 7: Remove the service
- In the manager node: ``docker service rm dotty``
- You can verify it is gone with ``docker service ls`` in the manager node, or ``docker ps`` in the worker nodes.

