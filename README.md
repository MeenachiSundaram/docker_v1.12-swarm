# docker_v1.12 - Swarm Tutorials

## Setting UP

- Three ubuntu machines in openstack under `Docker-Swarm` project

|Name|IP|
|:---:|:---:|
|manager1|192.168.0.52|
|worker-1|192.168.0.53|
|worker-2|192.168.0.54|

- Installing Docker-1.12

  1. sudo apt-get update
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
