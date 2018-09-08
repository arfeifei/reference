Docker: Remove all images and containers

#Problem:

You use Docker, but working with it created lots of images and containers. You want to remove all of them to save disk space.

#Solution:

Warning: This will destroy all your images and containers. It will not be possible to restore them!

Run those commands in a shell:
```
#!/bin/bash
# Delete all containers
docker rm $(docker ps -a -q)
# Delete all images
docker rmi $(docker images -q)
```
This solution has be proposed by GitHub user @crosbymichael in this [issue](https://github.com/moby/moby/issues/928#issuecomment-23538307)