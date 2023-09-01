## Underlying techs
- Mostly the **File system** management.  
Docker can be described as **having a external file system mounted on you local dir**, and then, you can use it like you own it.  
Also, the **kernal namespaces** which often utilized by OS is doing an amazing job here to isolate different processes.
- Docker images.  
Which easies the process to distribute the processes to others.

## Isolation utilities
- Kernal namespaces: Isolate process on **abstract** level.
- Control groups (cgroups): **Hardware** level. Manage the real resources (such as CPU, memory, disk I/O, and network bandwidth) shared by namespaces.