# Docker Pitfalls

### Connect i/o timeout

#### Error

`docker version` command complains

    FATA[0032] An error occurred trying to connect: Get https://192.168.59.103:2376/v1.16/version: dial tcp 192.168.59.103:2376: i/o timeout 
    
while `docker upgrade` shows that the client tne the server are of the same version (which means they should work together).

#### Diagnosis

The problem is because that I had created a host-only network `vboxnet0` before in an experiment of building a VM cluster using VirtualBox on my Mac mini, and `boot2docker init` would also create a boot2docker VM that uses `vboxnet0`.  However, boot2docker VM expects that the local network `vboxnet0` has IP prefix 192.168.53.xxx, whereas my `vboxnet0` has another IP prefix.

#### Solution

The solution is simple: 

1. Stop and destory boot2docker VM:

        boot2docker stop
        boot2docker destroy
        
1. Delete my `vboxnet0` local network from VirtualBox.

1. Recreate boot2docker VM and start it:

        boot2docker init
        boot2docker up
        docker version
        
### No space left on device

#### Error

When I run `docker build`, it complains not enough disk space.

#### Diagnosis

When above error happends with boot2docker, it means that the boot2docker VM, other than using has not enough disk space.

#### Solution

Clone the VMDK disk image created by `boot2docker init` into a VDI image, enlarge it, mount it to VM, and repartition it.  More details can be found [here](https://docs.docker.com/articles/b2d_volume_resize/).