# Docker Volumes

* [Containers and non-persistent data](#containers-and-non-persistent-data)
* [Containers and persistent data](#containers-and-persistent-data)

### Containers and non-persistent data
Many applications require a read-write filesystem in order to simply run – they won’t even run on a read-only filesystem. This means it’s not as simple as making containers entirely read-only. Every Docker container is created by adding a thin read-write layer on top of the read-only image

The writable container layer exists in the filesystem of the Docker host, and you’ll hear it called various names. These include local storage, ephemeral storage, and graphdriver storage. It’s typically located on the Docker host in these locations:
* Linux Docker hosts: /var/lib/docker/<storage-driver>/...
* Windows Docker hosts: C:\ProgramData\Docker\windowsfilter\...

This thin writable layer is an integral part of a container and enables all read/write operations. If you, or an application, update files or add new files, they’ll be written to this layer. However, it’s tightly coupled to the container’s lifecycle — it gets created when the container is created and it gets deleted when the container is deleted. The fact that it’s deleted along with a container means that it’s not an option for important data that you need to keep (persist).

### Containers and persistent data
Volumes are the recommended way to persist data in containers. There are three major reasons for this:
* Volumes are independent objects that are not tied to the lifecycle of a container
* Volumes can be mapped to specialized external storage systems
* Volumes enable multiple containers on different Docker hosts to access and share the same data

At a high-level, you create a volume, then you create a container and mount the volume into it. The volume is mounted into a directory in the container’s filesystem, and anything written to that directory is stored in the volume. If you delete the container, the volume and its data will still exist.

Third-party volume drivers are available as plugins. These provide Docker with seamless access external storage systems such as cloud storage services and on-premises storage systems including SAN or NAS.

All volumes created with the local driver get their own directory under /var/lib/docker/volumes on Linux, and C:\ProgramData\Docker\volumes on Windows. This means you can see them in your Docker host’s filesystem. You can even access them directly from your Docker host, although this is not normally recommended.