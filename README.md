# Apptainer in Docker

[![Docker images available at kaczmarj/apptainer](https://img.shields.io/badge/DockerHub-kaczmarj/apptainer-blue)](https://hub.docker.com/r/kaczmarj/apptainer)
[![Docker pulls](https://img.shields.io/docker/pulls/kaczmarj/apptainer)](https://hub.docker.com/r/kaczmarj/apptainer)
[![Docker pulls](https://img.shields.io/docker/pulls/kaczmarj/singularity)](https://hub.docker.com/r/kaczmarj/singularity)

The Dockerfile in this repository builds Apptainer. The resulting Docker image can be used on any system with Docker to build Apptainer images. This project is targeted towards high-performance computing users who have Apptainer/Singularity installed on their clusters but do not have Apptainer/Singularity on their local computers to build images.


**Note**: This project previously built Singularity.
See the [Linux Foundation post](https://www.linuxfoundation.org/press/press-release/new-linux-foundation-project-accelerates-collaboration-on-container-systems-between-enterprise-and-high-performance-computing-environments) regarding the name change to Apptainer.

## Convert local Docker image to Apptainer format

In the following example, we convert an existing Docker image to Apptainer format.

```bash
$ docker pull alpine:3.18.2
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/work \
    kaczmarj/apptainer build alpine_3.18.2.sif docker-daemon://alpine:3.18.2
```

This output `.sif` file will be owned by root, so you can change ownership:

```bash
sudo chown USER:GROUP alpine_3.18.2.sif
```

## Build Apptainer image in Docker

With the following command, we build a small Apptainer image defined in [`test_alpine.def`](test_alpine.def). This Apptainer image will be saved in the current directory `myimage.sif`.

```bash
$ docker run --rm --privileged -v $(pwd):/work kaczmarj/apptainer \
  build myimage.sif test_alpine.def
```

## Run Apptainer image in Docker

One can run a Apptainer image within this Docker image. This is not recommended, but it is possible.

```bash
$ docker run --rm --privileged kaczmarj/apptainer \
  run shub://GodloveD/lolcow
```

Here is the output:

```
INFO:    Downloading shub image
87.6MiB / 87.6MiB [======================================================================================================================================================] 100 % 1.4 MiB/s 0s
 _________________________________________
/ He that is giddy thinks the world turns \
| round.                                  |
|                                         |
| -- William Shakespeare, "The Taming of  |
\ the Shrew"                              /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

## Build image

Apptainer version 1.3.5:

```bash
$ docker build --build-arg APPTAINER_COMMITISH=v1.3.5 -t apptainer:1.3.5 .
docker build --build-arg APPTAINER_COMMITISH=v1.3.5 -t apptainer:1.3.5 -f Dockerfile .
```

Bleeding-edge (main branch):

```bash
$ docker build --build-arg APPTAINER_COMMITISH=main -t apptainer:latest .
```

## tranformer alphafold3 docker images to sif in slurm to run

```shell
docker build --build-arg APPTAINER_COMMITISH=v1.3.5 -t apptainer:1.3.5 -f Dockerfile .
docker pull brandonsoubasis/alphafold3:latest
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/work \
    apptainer:1.3.5 build alphafold3.sif docker-daemon://brandonsoubasis/alphafold3:latest
```

### test sif file

```shell
singularity exec --nv alphafold3.sif sh -c 'nvidia-smi'
singularity exec --nv alphafold3.sif <<args>>
singularity exec \
     --nv alphafold3.sif \
     --bind $HOME/af_input:/root/af_input \
     --bind $HOME/af_output:/root/af_output \
     --bind <MODEL_PARAMETERS_DIR>:/root/models \
     --bind <DATABASES_DIR>:/root/public_databases \
python alphafold3/run_alphafold.py \
     --json_path=/root/af_input/fold_input.json \
     --model_dir=/root/models \
     --db_dir=/root/public_databases \
     --output_dir=/root/af_output
# test apptainer in docker 
docker run --gpus all --rm --privileged -v $(pwd):/work apptainer:1.3.5 run /work/alphafold3.sif --nv sh -c 'nvidia-smi'
```
