# jlcross

# About [this repository](https://github.com/terasakisatoshi/jlcross)
Dockerfiles which build Julia (binary/docker-image) for (some) Arm devices e.g. Raspberry Pi Series (also including Zero) using cross compilation.

# How to use

- We will assume your build machine is
  - Ubuntu 16.04 equipped with high end Intel CPU.
  - macOS (Catalina) e.g. iMac

## Prepare environment

1. Install [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

For Ubuntu user, you will need install cross compilation tools:

```
$ apt-get update
$ apt-get install -y qemu-user-static binfmt-support
```

Thats' all

## Try to run Julia

To prove Julia can run on arm devices We've uploaded Docker images on [Docker Hub repository named `jlcross`](https://hub.docker.com/r/terasakisatoshi/jlcross)

- Install docker image from Docker Hub.

```console
$ docker pull terasakisatoshi/jlcross:rpizero-v1.3.0
Unable to find image 'terasakisatoshi/jlcross:rpizero-v1.3.0' locally
rpizero-v1.3.0: Pulling from terasakisatoshi/jlcross
(start to pull image...)

```

- Run Julia script using container

```console
$ cat hello.jl
using Pkg
Pkg.add("Example")
using Example
hello("World")
$ sudo docker run --rm -it -v $PWD:/work -w /work terasakisatoshi/jlcross:rpizero julia hello.jl
    Cloning default registries into `~/.julia`
    Cloning registry from "https://github.com/JuliaRegistries/General.git"
    Resolving package versions...     Added registry `General` to `~/.julia/registries/General`
 Resolving package versions...
 Installed Example ─ v0.5.1
  Updating `~/.julia/environments/v1.1/Project.toml`
  [7876af07] + Example v0.5.1
  Updating `~/.julia/environments/v1.1/Manifest.toml`
  [7876af07] + Example v0.5.1
  [2a0f44e3] + Base64
  [8ba89e20] + Distributed
  [b77e0a4c] + InteractiveUtils
  [56ddb016] + Logging
  [d6f4376e] + Markdown
  [9a3f8284] + Random
  [9e88b42a] + Serialization
  [6462fe0b] + Sockets
  [8dfed614] + Test
Hello, World
```

## Build Julia on your machine

- If you'd like to build Julia for your Raspberry Pi3, download, for example, `rpi3/Dockerfile-v1.3.0` and run the following command on your terminal to build Docker image which will include Julia binary:

```
sudo docker build -t jl4rpi3 -f Dockerfile-v1.3.0 .
```

- Please be patient, it will take long time (within a half day) to build.
- Also see my gist:
  - [(gist) Build Julia for RaspberryPi Zero](https://gist.github.com/terasakisatoshi/3f8a55391b1fc22a5db4a43da8d92c98)
  - [Cross Compile Julia For RaspberryPi3 using Docker](https://gist.github.com/terasakisatoshi/00fa7d7b81b7c6748f2298f6ff65bf6e)



## Get binary from container


- After building the docker image(or pulling docker image from our Docker Hub repository), you can get Julia binary by copying it from docker dcontainer generated by docker image what you've got

### Raspberry Pi Zero

- in this section, you'll know how to get Julia for Raspberry Pi Zero.

```console
# open your terminal e.g. your work station
$ cat get_binary.sh # write shell script by yourself like below:
#!/bin/bash

JL_VERSION=v1.3.0
IMAGE_NAME=terasakisatoshi/jlcross:rpizero-${JL_VERSION}
CONTAINER_NAME=jltmp_${JL_VERSION}
docker run --name ${CONTAINER_NAME} $IMAGE_NAME /bin/bash
docker cp ${CONTAINER_NAME}:/home/pi/work/julia-${JL_VERSION} .
docker rm ${CONTAINER_NAME}
$ bash get_binary.sh
$ ls
julia-v1.3.0
```

- Copy `julia-v1.3.0` to your Raspberry Pi zero:

```console
$ scp -r julia-v1.3.0 pi@raspberrypi.local:/home/pi
```

- After copying `julia-v1.3.0` to your Raspberry Pi, one need install the following dependencies via `apt` which is almost same as Dockerfile-1.3.0.

```console
# Open Your Raspberry Pi's terminal
$ sudo apt-get update && \
    apt-get install -y build-essential libatomic1 python gfortran perl wget m4 cmake pkg-config \
    libopenblas-dev \
    liblapack-dev \
    libgmp3-dev \
    libmpfr-dev
$ export 'PATH=/home/pi/julia-v1.3.0/bin:$PATH' >> /home/pi/.bashrc
$ source /home/pi/.bashrc
$ julia # Oh Yes!!!
```

That's all

## Jetson nano

- we will show another example:

```console
$ sudo docker create --name jltmp -it terasakisatoshi/jlcross:jetson-nano-1.1.1 /bin/bash
$ sudo docker cp jltmp:/home/jetson-nano/work/julia-1.1.1 .
$ sudo docker stop jltmp
$ docker rm jltmp
$ ls # you will see julia-1.1.1 current directory of your build machine
julia-1.1.1
```

That's all.

# Restriction

- We can't confirm building Julia version = `v1.2.0` on Raspberry Pi zero works fine.
  - You'll see some error message with respect to illegal instruction.
  - `v1.0.5`, `v1.1.1` and `1.3.0` is O.K.
- We can't confirm building Julia version = `v1.2.0` on Raspberry Pi3 using Docker its base image is `balenalib/raspberrypi3:buster-20191030` with error message something like:
  - ` undefined reference to llvm::BasicBlockPass::createPrinterPass(llvm::raw_ostream&, std::string const&) const'`


# References

- [Docker](https://www.docker.com/)
- [Docker Machine](https://docs.docker.com/machine/)
- [The easy way to set up Docker on a Raspberry Pi](https://medium.freecodecamp.org/the-easy-way-to-set-up-docker-on-a-raspberry-pi-7d24ced073ef)
- [How to install Docker on your Raspberry Pi](https://howchoo.com/g/nmrlzmq1ymn/how-to-install-docker-on-your-raspberry-pi)
- [Happy Pi Day with Docker and Raspberry Pi](https://blog.docker.com/2019/03/happy-pi-day-docker-raspberry-pi/)
- [Base Image List](https://www.balena.io/docs/reference/base-images/base-images-ref/)
- [multiarch/crossbuild](https://github.com/multiarch/crossbuild)

# References (Japanese blog)

- [Julia 1.1.0 をソースからビルドして RaspberryPi Zeroシリーズ や 3B+ とかで使いたいんじゃが？（Dockerでクロスコンパイルでどうじゃろ？）](https://qiita.com/SatoshiTerasaki/items/00f6bc2428ef81999164)
- [x86_64のUbuntuでC/C++のソースコードをARM/ARM64用にクロスコンパイルしてQEMUで実行する方法のまとめ](https://qiita.com/tetsu_koba/items/9bdcb59f912efbff3128)
- [ラズパイ向けのOpenCVを、x86_64機のDockerでビルド](https://qiita.com/mt08/items/51a2187076ddca0db7b0)
- [Jetson 上で Docker イメージをビルドするのが辛かったので EC2 上にビルド環境を作った](https://tech-blog.abeja.asia/entry/environment-of-building-docker-image-for-jetson)

# License

- As with all Dockerfiles, how to build julia, are licensed under the terms of MIT License.

- As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

  - Some additional license information which was able to be auto-detected might be found in [the repo-info repository's julia/ directory](https://github.com/docker-library/repo-info/tree/master/repos/julia).

  - As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
