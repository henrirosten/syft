# Evaluating syft nix support

## Syft
- https://github.com/anchore/syft
- https://github.com/anchore/syft/blob/main/DEVELOPING.md

Syft is:
"A CLI tool and Go library for generating a Software Bill of Materials (SBOM) from container images and filesystems."

At the time of writing this, syft does not properly support nix targets. However, there's a draft PR https://github.com/anchore/syft/pull/1107 that adds support for scanning nix-based docker images. 

This branch rebases the changes from PR https://github.com/anchore/syft/pull/1107 to current main to evaluate nix support in syft.

## Test nix docker image

```
# git-docker.nix: 
# (runs git in docker)
$ cat git-docker.nix 

{ pkgs ? import <nixpkgs> { }
, pkgsLinux ? import <nixpkgs> { system = "x86_64-linux"; }
}:

pkgs.dockerTools.buildImage {
  name = "git-docker";
  config = {
    Cmd = [ "${pkgsLinux.git}/bin/git" ];
  };
}
```

```
# Build the docker image
$ nix-build git-docker.nix
```

```
# Load the docker image
$ docker load < result
```

```
# List images to check the REPOSITORY and TAG:
$ docker images
REPOSITORY             TAG                                IMAGE ID       CREATED        SIZE
git-docker             lpal3d7lxf0y8mmambj1q0ilhnq6llyv   6aaab43d70a9   52 years ago   289MB
```

```
# Run syft to generate (runtime) cyclonedx SBOM:
$ go run cmd/syft/main.go docker:git-docker:lpal3d7lxf0y8mmambj1q0ilhnq6llyv -o cyclonedx-json > git.cdx.json
```