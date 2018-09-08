Docker Swarm Mode

![docker-swarm-1](/blog/content/images/2017/10/docker-swarm-1.png)
## Command
> docker commonly used command in swarm mode
###1.Start & join Swarm
```
docker swarm init
docker swarm join-token manager
```
###2.Service operation
```
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service
```
## Run Visualizer GUI on swarm mode
> http://mydockerhost:9000/
>
###1. Create service name as ***visualizer***
```
docker service create \
  --name=visualizer \
  --publish=9000:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```
###2. Scale service ***visualizer*** as ***3*** replica
```
docker service scale visualizer=3
```
![docker_visualizer](/blog/content/images/2017/10/docker_visualizer.jpg)
###3. Check ***visualizer*** run time
```
docker service ps visualizer
```
###3. Delete/stop ***visualizer*** service
```
docker service rm visualizer
```
## Deploy your application build by docker-compose.yml
```
docker stack deploy -c docker-compose.yml your-app-name
```
You can scale the app by changing the ***replicas*** value in docker-compose.yml, saving the change, and re-running the docker stack deploy command:
> docker-compose.yaml example:
> 
```
verision '3'
services:
  mysrv1:
    image:myimg1
  deploy:
    replicas:3
    restart_policy
      condition:on-failure
    resources:
      limites:
        cups:"0.1"
        memory: "50M"
  ports:
    - 8080:80
...
```
## Take down the app and the swarm
```
docker stack rm your-app-name
```
## Docker Swarm Demo Video
>From [https://www.youtube.com/watch?v=1hLiqVkj3K8](https://www.youtube.com/watch?v=1hLiqVkj3K8)
[Reference: https://docs.docker.com/get-started/part4](https://docs.docker.com/get-started/part4)