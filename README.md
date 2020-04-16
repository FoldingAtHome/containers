# Official Folding@home Containers

The official Folding@home containers are designed to be simple and run
in any container environment - desktop, laptop, Kubernetes, Helm,
Docker Compose, OpenShift, Cloud...

Containers track stable versions of the Folding@home Client. They will
run on any Linux distribution, and are based on LTS version of Ubuntu,
OpenCL, and CUDA libraries. They contain enough utilities to exec in and be
able to debug any problems.

## Containers

* GPU container - [fah-gpu](fah-gpu/)

## Deployments

This repo will also container Helm templates and other deployment
scripts/tools for a variety of environments.

## Folding@home Container Design Rules

* Containers will mount all read-write state, including config.xml
  which also has client state, into `/fah/`.
* Containerized clients will output to stdout/sterr for container logs.
* Containers are designed to be monitored via logs, and controlled with files.
* No example configurations will expose ports, and that functionality will
  be configured off in examples. Folding@home was built to run on
  desktops/LANs, and currently should not be exposed to the internet.

## Tags

* `MAJOR.MINOR.PATCH` - follows Folding@home Client version.
* `MAJOR.MINOR.PATCH-rc...` - test builds of the containers, stable clients.
* `latest` - please never use latest in production, specify a version. Points
  at the latest released container.

## Folding@home Websites

* Folding@home: https://foldingathome.org/
* Folding@home Support Forum: <https://foldingforum.org/>
* Folding@home Containers GitHub: <https://github.com/foldingathome/containers/>
* Folding@home Docker Hub: <https://hub.docker.com/u/foldingathome>
