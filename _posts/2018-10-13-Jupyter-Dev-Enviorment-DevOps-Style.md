---
layout: post
title: "Jupyter Dev-Enviorment DevOps Style"
author: "Ben Engel"
categories: devop
tags: 
image: 
---

I often just want to spin my jupyter notebook server to test some ideas. But with more and more machines its hard to keep a clean and relaiable enviorment on all of them. So I decided to use my all beloved docker stack to the rescue. 

## The Docker Jupyter Notebook Images
As long as you are looking for a CPU bound container (more on GPU containers in an later article) I just can recommend you the official [jupyter stack](https://github.com/jupyter/docker-stacks). These are a bunch of herachical organized images which expand on another. The `jupyter/scipy-notebook` image is the first one with the entire python data science stack in it, but I'm trying to dig more int spark recently so I use the `jupyter/all-spark-notebook` for ease of use.



But to give you an overview here the entire hierarchy in a diagram

![](../assets/img/jupyter_stack.svg)

## Customize The Image

One way of customization could be to add a couple of modules you need frequently to the image by building a custom image on to of the jupyter stack one. 

This could look like this

```bash
#Dockerfile
FROM jupyter/minimal-notebook
WORKDIR /opt
COPY requirements.txt /opt/requirements.txt
RUN pip install -r /opt/requirements.txt
EXPOSE 8888
ENTRYPOINT jupyter-notebook --no-browser --ip='0.0.0.0' --port=8888 --allow-root --NotebookApp.token=''
```

Here we: 

- fist select the image to build up on `jupyter/minimal-notebook`,

- changing the working directory to `/opt`

- copy over the requirements.txt
- installing the listed package with pip
- exposing the port 8888 so we can forward him during docker run to our host
- starting the notebook server without a password requirement  

And the corresponding `requirements.txt`

```bash
jupyter
pandas
seaborn
matplotlib
scikit-learn
```

If we want to build the image we put both files in an empty folder and run

```bash
docker build . -t IMAGENAME
```

within the folder. This builds the image locally so we can just can use `docker run`  like 

```bash
docker run -d -p 8888:8888 IMAGENAME
```

We also can bind mount our project folder to the container to worke with the instance like we do locally

```bash
docker run -d -p 8888:8888 -v /path/to/project/folder:/home/jovyan/work IMAGENAME
```



## Make It Handy

To not always retyping the commands and using `docker ps` to refind our instance we use the `--name` flag of docker 

```python
docker run -d -p 8888:8888 -v /path/to/project/folder:/home/jovyan/work --name jnb IMAGENAME
```

the next step is to define alias to init,start, stop and update our environment

```bash
alias jnb_init='sudo docker run -d -v "/path/to/project/folder:/home/jovyan/work" -p 8888:8888 --name jnb jupyter/all-spark-notebook start-notebook.sh --no-browser --ip="0.0.0.0" --port=8888 --allow-root --NotebookApp.token=""
--NotebookApp.notebook_dir="/home/jovyan/work"'
alias jnb_start='sudo docker start jnb'
alias jnb_stop='sudo docker stop jnb'
alias jnb_remove='jnb_stop && sudo docker rm jnb'
alias jnb_update='jnb_remove && jnb_init
```



So we can initiate our notebook server with `jnb_init` and reach it under `localhost:8888` .

To stop it we just run `jnb_stop` and if there is a new image we can update our environment with `jnb_update`.



Thats it for now, so long.