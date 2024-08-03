# 17. Docker install and config

General info and config on running containers
```
docker info
```

Seeing all the commands
```
docker
```
- Docker command format: 
	- **Management commands**: `docker <command> <sub-command> (options)`
	- **Old way**: `docker <command> (options)` (still works)
	- e.g. `docker container run` vs `docker run` both work

# 18. Starting an Nginx web server

- Docker run creates a new container each time

```
docker container run --publish 80:80 nginx
```
1. Downloaded an image for `nginx`
2. Started a new container for that image
3. Opened port 80 on the host IP
4. Routes the traffic to the container IP port 80
- You'll get a bind error if the host port is already used

To do a detached version, with name specified (otherwise name is random):
```
docker container run --publish 80:80 --detach --name webhost nginx
```

To view logs in detached container:
```
docker container logs webhost
```

View running nginx processes
```
docker container top webhost
```

Remove all containers (`-f` to force remove running containers, `-q` to only list IDs)
```
docker container rm -f $(docker ps -aq)
```

# 19. What happens when we run a container
What happens in `docker container run`?
- Looks for image locally in image cache
- If not found, looks in remote image repository (defaults to Docker Hub)
- Downloads the latest version by default (`nginx:latest`)
- Creates a new container based on that image and prepares to start
- Gives it a virtual IP on a private network inside Docker Engine
- Opens up port 80 on host and forwards to port 80 on container (this was only because we provided `-p` option)
- Starts container by using the CMD in the image's Dockerfile

# 20. Container vs VM
Containers **aren't** mini-VMs
- They're just processes
- Limited to what resources they can access
- Exit when the process exits

For example, you can see the processes running inside a container (`docker top`) using `ps aux`
- `docker top` even shows the PID

# 24. What's going on in containers?