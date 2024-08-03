Categorized by management command

**Notes**:
- `CONTAINER` seems to accept either ID or name
- Use `--help` for context on any command or just type the management command, e.g. `docker container` 
- You can also use tab completion on certain commands that require `CONTAINER` arguments (`rm`, `stop`, etc.)
### container

Run an image (creates container)
- Pulls image automatically if it doesn't exist
```
docker run
```
- `-d` detached
- `-p` publish a container's ports to the host (`host:container`)
- `-e` pass environment variable into container

List all running containers
-  Management command also accepts `ls`
```
docker ps
```
- `-a` to show all running and stopped containers
- `-q` only list IDs (useful when deleting)

Stop a container
```
docker stop CONTAINER
```
- Only requires enough digits of container ID to make it unique

Show logs
```
docker logs CONTAINER
```
- `-f` to follow along

Show running processes in container
```
docker top CONTAINER
```

Remove containers
```
docker rm CONTAINER
```
- `-f` force (removes running containers too)
- Can provide a list of containers, or use `$(docker ps -aq)`