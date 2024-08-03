# 1. Docker 2023
### Build, ship, run
The same 3 key concepts have existed in Docker since its invention:
1. Docker image (**build**)
	- Universal app packager (run everywhere and on anything)
	- **Dockerfile**: Set of instructions to make a container (or Docker) image
		- Each command in a Dockerfiles will create layers (tarballs)
	- `docker build` builds an image
2. Docker registry (**ship**)
	- Universal app distribution (sharing container images)
	- Each image has a unique SHA hash
	- `docker push` pushes images to a registry
	- `docker pull` pulls an image to your image
3. Docker container (**run**)
	- Identical runtime environments
	- Requires Docker engine on your system for orchestration
	- `docker run` runs the app in a container
	- The container runs in a **namespace** that is isolated it from the rest of the operating system (own IP address, process list, etc.)

### Definitions
- **Image** is the application (binaries, libraries, source code)
- **Container** is an instance of an image running as a process
	- Many contains can run the same image
# 2. Quick Container run
https://labs.play-with-docker.com/

Details about Docker client and sever:
```
docker version
```
- Docker is a two part system:
	- Client (CLI): where you type your commands
	- Server: the docker daemon
- **Docker Engine** encompasses both of these 
- Communicate via socket, TCP/TLS or SSH

Run a container with an HTTP server:
```
docker run -d -p 8800:80 httpd
```
- `-d` - detached
- `-p` - opens a port and publishes it on host IP
	- 8800 - port for listening to on host
	- 80 - port for listening to apache on container
- `httpd` - the image we want to run
- `curl localhost:8800` to verify it works

See all running container processes
```
docker ps
```

### Linux concepts
Docker takes advantage of Linux features like:
- namespaces
- cgroups
- veth
- iptables
- union mount

# 3. Why Docker?
1. Isolation
	- All apps use to share the same filesystem on a server (servers become brittle)
	- Then came VMs, abstract out applications into smaller VMs all writing on same host (hard to manage so many VMs each with their own operating systems)
2. Environments
	- Fixes "works on my machine" problem
3. Speed
	- Faster to develop, test, etc.
