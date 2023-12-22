## Underlying techs
- Mostly the **File system** management.  
Docker can be described as **having a external file system mounted on you local dir**, and then, you can use it like you own it.  
Also, the **kernal namespaces** which often utilized by OS is doing an amazing job here to isolate different processes.
- Docker images.  
Which easies the process to distribute the processes to others.

## Isolation utilities
- Kernal namespaces: Isolate process on **abstract** level.
- Control groups (cgroups): **Hardware** level. Manage the real resources (such as CPU, memory, disk I/O, and network bandwidth) shared by namespaces.

### Layers
Every command line in a `Dockerfile` represents a layer which is physically separated using **UnionFS** technique.  
Layer mechanism is mostly for inter-container convinience, making it possible to efficiently manage and share file systems while minimizing storage overhead.

## Persist the DB
[How to persist the data](https://docs.docker.com/get-started/05_persisting_data/)
Any changes in one container cannot be seen in the other.  
Contaner volumes can fix this.  
- Volume mounts: Create a volume, and attach(mount) it to your host directory. Remember the name of the volume.
- Bind mounts: Bind the host directory (local source code) to the container, which makes it capable of managing the file through your host.  
   Enter your local corresponding file, run the commands:
   ```
   docker run -dp 127.0.0.1:3000:3000 `
    -w /app --mount "type=bind,src=$pwd,target=/app" `
    node:18-alpine `
    sh -c "yarn install && yarn run dev"
   ```
   The commands include downloading dependencies.

| | Named volums | Bind mounts|
:--|:--|:--
Host location | Docker chooses | You decide
Mount example(using `--mount`) | `type=volume,src=my-volume,target=/user/local/data` | `type=bind,src=/path/to/data,taget=/usr/local/data`|
Populates new volume with container contents| Yes | No
Supports Volume Drivers | Yes | No

## Multi-container apps
### Communicate with network
- Create a network.
- Attach the containers to the network.

### Use docker compose
- Create `compose.yaml` file.
   You need to create volumes manually.
- run command `docker compose -d`, in which `-d` means run in dettach mode.