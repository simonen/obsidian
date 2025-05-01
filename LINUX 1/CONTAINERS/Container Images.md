
Container images are created in the Docker format

#### Using Registries

Container registries are specified in:
**/etc/containers/registries.conf**

A user running rootless containers can create a file, which in case of conflict, will override the main file in etc:
**~/.config/containers/registries.conf**

To get info about the current environment
**$ podman info**

To get info about currently used registries:
**$ podman info | grep -A 10 registries**

#### Finding Images

Search for an image:
**$ podman search** [registry optional] *IMAGE_NAME*

To search for images using filters:
**$ podman search --filter is-official=true** *NAME*
**$ podman search --filter stars=5** *NAME*

**UBI**: Universal Base Image. The image used for every Red Hat product

#### Inspecting Images

To inspect images directly from the registry:
**$ skopeo inspect docker**://*IMAGE_URL*

To inspect images locally downloaded:
**$ podman inspect**

To get a list of local images:
**$ podman images**

To pull an image. Recommended way of pulling and inspecting before running:
**$ podman pull** *NAME*

#### Building Images From Containerfiles

Containerfile is standardized version of Dockerfile

sample Containerfile:
```
FROM docker.io/library/alpine
RUN apk add nmap
CMD ["nmap", "-sn", "172.16.0.0/24"]
```

To build an image from the Containerfile:
**$ podman build -t** *CONTAINER_NAME:TAG /CONTAINERFILE_DIR*

**tag**: means version
. : the current directory, if it contains the Containerfile
