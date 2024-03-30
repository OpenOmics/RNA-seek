## Steps for Building Docker Images

Directly below are instructions for building a base image for the `RNA-SEEK DEPENDENCIES` using the provided Dockerfile:

```bash
# See listing of images on computer
docker image ls

# Build from Dockerfile
docker build --no-cache -f Dockerfile --tag=rna_seek:v0.1.0 .

# Testing, take a peek inside
docker run -ti rna_seek:v0.1.0 /bin/bash

```

### Other Recommended Steps

Scan your image for known vulnerabilities:

```bash
docker scan rna-seek:v0.1.0
```