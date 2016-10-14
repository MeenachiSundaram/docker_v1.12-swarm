# docker_v1.12 - Swarm Tutorials

## Setting UP

- Three ubuntu machines in openstack under `Docker-Swarm` project

|Name|IP|
|:---:|:---:|
|manager1|192.168.0.52|
|worker-1|192.168.0.53|
|worker-2|192.168.0.54|

- Installing Docker-1.12

  - sudo apt-get update
  - sudo apt-get install apt-transport-https ca-certificates
  - sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
  - Add an APT-source entry
    - vim /etc/apt/sources.list.d/docker.list
    - Add an entry `deb https://apt.dockerproject.org/repo ubuntu-trusty main`

      (or)

    - sudo echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
  - sudo apt-get update
  - To purge existing docker
    - sudo apt-get purge lxc-docker
  - To verify that APT is pulling from the right repository
    - apt-cache policy docker-engine
  - Prerequisites by Ubuntu Version
    - sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
    - sudo reboot
  - sudo apt-get update
  - sudo apt-get install docker-engine
  - Check the docker version
    - docker -v
  - sudo service docker start

## Creating a Docker-Swarm visualizer

we gonna use docker `manomarks/visualizer` image to get a clear idea about docker swarm.

- Run the following command on the master node

  ```
  docker run -it -d -p 5000:5000 -e HOST=192.168.0.52 -e PORT=5000 -v /var/run/docker.sock:/var/run/docker.sock manomarks/visualizer
  ```
  ![b](./readme-images/b.png)
- `docker ps` shows the running *manomarks/visualizer* on port *5000*

  ![a](./readme-images/a.png)

- Visit **http://192.168.0.52:5000** in browser to see the running docker-visualizer.

## Creating Swarm Master

- On master1 node run the following command
    ```
    docker swarm init --advertise-addr 192.168.0.52
    ```

    ![1](./readme-images/1.png)

- In visualizer you can see the master node

  ![2](./readme-images/2.png)

## Attaching Worker Nodes to Swarm Master

- On worker-1 node run the following `docker swarm join` command with the token generated on `master1`
  ```
  docker swarm join \
  --token SWMTKN-1-3kjq5jkbv2tfxzr5kkvtbqmq7eboj1mkewr47g7dhpo69gfnsh-4ja77pbzmvgonnzvq8aio6xku \
  192.168.0.52:2377
  ```

  ![3](./readme-images/3.png)

- In visualizer you can see the worker-1 node

  ![4](./readme-images/4.png)

- Also on worker-2 node run the same `docker swarm join` command
  ```
  docker swarm join \
  --token SWMTKN-1-3kjq5jkbv2tfxzr5kkvtbqmq7eboj1mkewr47g7dhpo69gfnsh-4ja77pbzmvgonnzvq8aio6xku \
  192.168.0.52:2377
  ```

  ![5](./readme-images/5.png)

- In visualizer you can see the worker-2 node

  ![6](./readme-images/6.png)

### List of Nodes

Now in master run the below command to see the list of nodes in the swarm.
  ```
  docker node ls
  ```

  ![7](./readme-images/7.png)

## Deploy a service to the swarm

- On Manager node run the following command to deploy a [sample voting service](https://hub.docker.com/r/instavote/vote/) for testing the swarm usage

  ```
  docker service create --replicas 1 --name vote -p 8080:80 instavote/vote
  ```

  - Here we use `docker service create` to run a service in docker swarm
  - `--replicas 1` denote the number of running instances
  - `-p 8080:80` used for port mapping
  - `instavote/vote` is a container we used

- To see the list of running services, Run `docker service ls`

  ![8](./readme-images/8.png)

## Inspecting the running service

- Run `docker service inspect --pretty <SERVICE-ID>` to display the details about a service in an easily readable format.

  ![11](./readme-images/11.png)
- Running `docker service inspect <SERVICE-ID>` without *--pretty* will display the details in **JSON** format
  ```
  root@manager1:/home/ubuntu# docker service inspect 6oxnfhi8o5lh
  [
      {
          "ID": "6oxnfhi8o5lh02ktc5gjarjk2",
          "Version": {
              "Index": 42
          },
          "CreatedAt": "2016-10-14T13:23:10.799339539Z",
          "UpdatedAt": "2016-10-14T13:23:10.830455894Z",
          "Spec": {
              "Name": "vote",
              "TaskTemplate": {
                  "ContainerSpec": {
                      "Image": "instavote/vote"
                  },
                  "Resources": {
                      "Limits": {},
                      "Reservations": {}
                  },
                  "RestartPolicy": {
                      "Condition": "any",
                      "MaxAttempts": 0
                  },
                  "Placement": {}
              },
              "Mode": {
                  "Replicated": {
                      "Replicas": 1
                  }
              },
              "UpdateConfig": {
                  "Parallelism": 1,
                  "FailureAction": "pause"
              },
              "EndpointSpec": {
                  "Mode": "vip",
                  "Ports": [
                      {
                          "Protocol": "tcp",
                          "TargetPort": 80,
                          "PublishedPort": 8080
                      }
                  ]
              }
          },
          "Endpoint": {
              "Spec": {
                  "Mode": "vip",
                  "Ports": [
                      {
                          "Protocol": "tcp",
                          "TargetPort": 80,
                          "PublishedPort": 8080
                      }
                  ]
              },
              "Ports": [
                  {
                      "Protocol": "tcp",
                      "TargetPort": 80,
                      "PublishedPort": 8080
                  }
              ],
              "VirtualIPs": [
                  {
                      "NetworkID": "5t5bl9qk4x8c9oy549xr1j0lb",
                      "Addr": "10.255.0.6/16"
                  }
              ]
          },
          "UpdateStatus": {
              "StartedAt": "0001-01-01T00:00:00Z",
              "CompletedAt": "0001-01-01T00:00:00Z"
          }
      }
  ]
  ```

- Run `docker service ps <SERVICE-ID>` on master to see which nodes are running the service
  ```
  docker service ps 6oxnfhi8o5lh
  ```
  ![12](./readme-images/12.png)

- Also we can run `docker ps` on the node where the task is running to see details about the container for the task.

  ![13](./readme-images/13.png)

- In visualizer we can see the created service and the node in which it is running.

  ![9](./readme-images/9.png)

- By visiting http://<IP>:8080 in browser we can see the output with a container ID.

  - `IP` could be any of master or worker IP's, since it is a swarm service all will be redirected to the same.

  - `8080` is the port that we specified while creating the service.

  ![1.gif](./readme-images/gif/1.gif)

## Service Scalling in swarm

- In master node running the command `docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>` will change the desired state of running service in swarm.
  ```
  docker service scale 6oxnfhi8o5lh=6
  ```
- Run `docker service ls` to see the current state of **Replicas**
  ![10](./readme-images/10.png)
- Run `docker service ps <SERVICE-ID>` will display the task node list on which it is running.
  ```
  docker service ps 6oxnfhi8o5lh
  ```
  ![15](./readme-images/15.png)
- In visualizer you can see the detailed view about the number of running services and on various nodes.
  ![14](./readme-images/14.png)
- By visiting http://<IP>:8080 in browser we can see the output with a various container ID's.

  - `IP` could be any of master or worker IP's, since it is a swarm service all will be redirected to the same.

  - `8080` is the port that we specified while creating the service.

  ![2.gif](./readme-images/gif/2.gif)

## Removing Swarm Services

- Run  `docker service rm <SERVICE-ID>` will remove the complete swarm service along with Replicas.
  ```
  docker service rm 6oxnfhi8o5lh
  ```
  ![16](./readme-images/16.png)

## Apply rolling updates to a service

[Link](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/)

## Drain a node on the swarm

[Link](https://docs.docker.com/engine/swarm/swarm-tutorial/drain-node/)
