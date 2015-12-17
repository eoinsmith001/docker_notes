# Run through of Docker

- on osx, even if the docker daemon is running, new terminals won't be able to use docker until they execute `eval $(docker-machine env default)`
- Use a Dockerfile to build an image, usually by using FROM and then incrementing
- Start a registry on localhost with `docker run -d -p 5000:5000 --restart=always --name registry registry:2`
- Build from a directory containing a Dockerfile, and use a tag to point it at a registry/repository `docker build -t localhost/namesvc .`
- Tag an existing image in order to point it at a specific repository
- official docker tag syntax: `Usage: docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]`
- This tags an existing image with `docker tag localhost/namesvc localhost:5000/namesvc`
- docker push pushes an image OR repository to the registry
- This works once a localhost registry is running: `docker push localhost:5000/namesvc`
- if an image exists in multiple repos (i.e. same image id, with different repo values in `docker images`), docker rmi image_id will complain, but docker rmi <repo_value> will untag the image and remove the entry from docker images
- if in `docker images`, the `<repo_value>` is a valid reachable registry, you can do `docker push <repo_value>`
- if there is only one image id, docker rmi <image_id> will remove it
- if the image is removed, `docker pull localhost:5000/namesvc` will retrieve it from registry
- actually applying a TAG which appears in docker images command is `docker tag localhost:5000/namesvc localhost:5000/namesvc:v0` (the colon at the end)
- can push a tagged image separately to the registry `docker push localhost:5000/namesvc:v0`
- Can we search our private registry?  No, there are some threads about this, but it boils down to how one would search all possible storage backends...
- rerun the registry with a specific config file: `docker run -d -p 5000:5000 --restart=always --name registry -v `pwd`/config.yml:/etc/docker/registry/config.yml registry:2`
- curl private local registry on osx?  Use the ip of `DOCKER_HOST`: `curl -X GET 192.168.99.100:5000/v2/_catalog` produces `{"repositories":["namesvc"]}`
- then for tags: `curl -X GET 192.168.99.100:5000/v2/namesvc/tags/list`: `{"name":"namesvc","tags":["v0"]}`
- `docker history <image_id>` to see how the image was constructed


## Develop live against a running container

- get a basic Dockerfile (oneliner) which provides a reasonable dev base (eg `FROM nodesource/node:4.2`)
- build it as an image `docker build -t localhost:5000/ndev:v0 . ` (Here the built image is tagged as being part of my local registry, and as v0)
- Optional: push to private registry (if one is running): `docker push localhost:5000/ndev:v0`
- Optional: validate presence in private registry: `curl -X GET 192.168.99.100:5000/v2/_catalog`, `curl -X GET 192.168.99.100:5000/v2/ndev/tags/list` (The weird ip is the ip of `$DOCKER_HOST`, as this was done on OSX)
- run an interactive shell (`-i`), volume mapping your checkout of repo to, say, `/src/app`
- `docker run -ti -v /Users/esmith/Node/nock_try/namesvc:/src/app localhost:5000/ndev:v0 /bin/bash`
- navigate to `/src/app` in the interactive container session; all the code is there!
- run tests in the container
- edit code on localhost, container code updates automatically
- or (possibly better), run tests in one-shot containers as `docker run -ti -v /Users/esmith/Node/nock_try/namesvc:/src/app localhost:5000/ndev:v0 /src/app/node_modules/.bin/mocha /src/app/test`
- For some reason using the dot notation for $CWD does not seem to work with -v?
