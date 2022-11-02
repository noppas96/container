# Storage

Docker uses storage drivers to store image layers, and to store data in the writable layer of a container, The container’s writable layer does not persist after the container is deleted

## Container Size on Disk
To view the approximate size of a running container
```
docker ps -s
```
 - size: the amount of data (on disk) that is used for the writable layer of each container.
 - virtual size: the amount of data used for the read-only image data used by the container plus the container’s writable layer size. 
 
    *Multiple containers may share some or all read-only image data. Two containers started from the same image share 100% of the read-only data, while two containers with different images which have layers in common share those common layers. Therefore, you can’t just total the virtual sizes. This over-estimates the total disk usage by a potentially non-trivial amount.*

The ways a container can take up disk space

- Disk space used for log files stored by the logging-driver. by default, Docker captures the standard output (and standard error) of all your containers, and writes them in files

- Volumes and bind mounts used by the container.

## The copy-on-write (CoW) strategy
When you use docker pull to pull down an image from a repository,  Docker’s local storage area, which is usually /var/lib/docker/ on Linux hosts, list the contents of /var/lib/docker/<storage-driver\>. This example uses the overlay2 storage driver 
[more.](https://docs.docker.com/storage/storagedriver/)

## OverlayFS Storage Driver

OverlayFS layers two directories on a single Linux host and presents them as a single directory. These directories are called layers and the unification process is referred to as a union mount. OverlayFS refers to the lower directory as lowerdir and the upper directory a upperdir. The unified view is exposed through its own directory called merged.

<p align="center"><img src="./images/fig1.jpg"></p>

To create a container, the overlay driver combines the directory representing the image’s top layer plus a new directory for the container. The image’s top layer is the lowerdir in the overlay and is read-only. The new directory for the container is the upperdir and is writable.

The **upper** directory contains the contents of the container’s read-write layer, which corresponds to the OverlayFS upperdir.

The merged directory is the union mount of the **lowerdir** and **upperdir**, which comprises the view of the filesystem from within the running container.

The work directory is internal to OverlayFS.

### Reading files

- The file does not exist in the container layer: it is read from the image (lowerdir).

- The file only exists in the container layer: file exists in the container (upperdir) and not in the image (lowerdir), it is read directly from the container.

- The file exists in both the container layer and the image layer: Files in the container layer (upperdir) obscure files with the same name in the image layer (lowerdir).

### Modifying files or directories



