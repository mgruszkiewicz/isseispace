---
title: "Build OCI image with artefact"
date: 2025-01-01T20:41:14+01:00
draft: true
---

# What are OCI artifact images

> OCI artifacts are a way of using OCI registries, or container registries that are compliant with specifications set by the Open Container Initiative, to store arbitrary files.
https://edu.chainguard.dev/open-source/oci/what-are-oci-artifacts/

## Requirements

This writeup will be using Oras as a utility to build artifacts, and Kubernetes 1.31 as a example how can it be mounted as additional, read-only volume in pod.

For testing, i will be using a local [`zot` registry](https://github.com/project-zot/zot), running in Docker.
```
docker run -d -p 5000:5000 --name zot ghcr.io/project-zot/zot-linux-amd64:v2.1.1
```

# How to build a OCI Artifact

## Artifact type

https://github.com/opencontainers/artifacts/blob/main/artifact-authors.md#defining-a-unique-artifact-type

## Building first OCI Artifact


```
❯ oras push docker.io/mgruszkiewicz/whisper.cpp:models \
    --artifact-type application/vnd.isseispace.whisper.models \
    *.bin
✓ Uploaded  ggml-base-q5_1.bin                                                                                                                     56.9/56.9 MB 100.00%    19s
  └─ sha256:422f1ae452ade6f30a004d7e5c6a43195e4433bc370bf23fac9cc591f01a8898
✓ Uploaded  application/vnd.oci.empty.v1+json                                                                                                            2/2  B 100.00%     1s
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  application/vnd.oci.image.manifest.v1+json                                                                                               609/609  B 100.00%     0s
  └─ sha256:b94c5e8483c42dad7483cab9b227ed939f77be7e311beeb251c585575c924e04
Pushed [registry] docker.io/mgruszkiewicz/whisper.cpp:models
ArtifactType: application/vnd.isseispace.whisper.models
Digest: sha256:b94c5e8483c42dad7483cab9b227ed939f77be7e311beeb251c585575c924e04
```


For building a OCI image that contains all files in current directory, you can use wildcard `*`
```
oras push --plain-http localhost:5000/whisper:v3 \
    --artifact-type application/vnd.isseispace.whisper.config \
    *
```

# How to use a OCI Artifact

Kubernetes 1.31 added a support for mounting OCI artifacts as read-only volumes, but to use it, **you need to enable `ImageVolume` feature gate**
https://kubernetes.io/blog/2024/08/16/kubernetes-1-31-image-volume-source/

Additionally, you need runtime which supports OCI artifacts. At the time of writing, i think only `CRI-O` supports this.



# Compatibile registries
Pretty much all "mainstream" registries support OCI images. You can learn more on Oras [Compatible OCI Registries](https://oras.land/docs/compatible_oci_registries) page 

Please note, that you won't be able to pull a OCI artefact image using `docker`
