---
Title: Docker Primer
Author: /u/radakul
tags: [normal]
---
___

## Introduction

This guide will provide a cursory, high-level overview of Docker. This is not intended to replace the [official Docker documentation](https://docs.docker.com/), but is instead meant to distill the contents into an easy-to-read format, specifically aimed at self hosters.

Docker is available on all Windows, Mac and Linux, but is used most often on Linux machines.  Both Mac and Windows have a graphical interface called [Docker Desktop](https://www.docker.com/products/docker-desktop/) which can be used to interface with containers. Recently, DD4L (Docker Desktop for Linux) was made public, unifying the experience across all operating systems. Please note that DD4L runs a VM in the background, so it may be more resource-heavy, but is good for learning the basics.

In most deployments, especially in the Enterprise space, Docker (and similar containerization solutions) will be run on Linux servers, so it's useful to get some practice with the command line. This guide will provide common command-line commands for managing Docker containers, volumes, images and networks.

For those who are more comfortable with a graphical experience, great solutions such as [Portainer](https://portainer.io) exist, which integrate with the Docker engine and allow for a more robust and user-friendly way to manage a Docker installation.

Enough intro, let's get started!

### Requirements

- Basic understanding of virtualization
- An operating system which supports Docker
- A willingness to learn and make mistakes!

### Installation

We'll keep this section simple - follow the [official installation documents](  https://docs.docker.com/get-docker/) for whichever operating system you are using. The author of this document is using Linux, so any included screenshots will be from a Linux terminal.

### Basics & Methodology

Docker is a *containerization* engine and platform. What does this mean? Rather than needing to install an entire operating system for a specific application and manage all the dependencies (specific programming libraries, security updates, version updates, etc.), I can instead package *only* what is needed to keep that application running and place it in a "container". This way, I can tightly control what dependencies exist and ensure things don't break.

In addition, just like a shipping container can fit on a semi truck (or lorry, for the folks not based in the US) or a large ship, a Docker container can be packaged and run on any system that has Docker installed. And since Docker is cross-platform, this ensures ultimate compatibility across all operating systems.

### Docker Structure

Docker is organized in a hierarchical fashion:

- A **Dockerfile** is used to specify what commands to run, what to install, etc. This is exactly like what would be done via the command line but is instead in a text file
- A Docker **image** is then constructed from the Dockerfile. The images are built in **layers**, with each command inside the Dockerfile resulting in a new image layer.
    - A downside to this layer system is you have to structure your Dockerfile to make sense with how the layers will operate - layers are constructed into the image and then deleted, so you have to plan how the commands will link together. You may not have access to a previous layers' output without proper planning.
    - Images can be tagged to differentiate versions, i.e. using &lt;imagename&gt;:&lt;v1.0&gt; instead of the default of &lt;latest&gt;.
- One or more Docker **containers** can then be created from an image
    - Unless otherwise configured, Docker containers are ephemeral - changes you make won't survive a reboot of the container.
    - To the point above, Docker containers don't have access to storage or other persistent settings unless explicitly defined. The **VOLUME** option is used to specify the mount points that will be persistently available.
    - Unless otherwise specified, containers are *ephemeral*, or designed to be operated in a "run and done" type fashion - that is, a container is started, executes a command (defined in the **CMD** portion of it's Dockerfile) and then exits.
- Given the hierarchical nature, it's important to note that an **image** *cannot* be deleted if a **container** is using it. Containers need to be stopped and deleted in order to remove an image that is dependent on it.
    - This could be forced but probably isn't a good idea in a production environment

### Basic command cheatsheet

Let's summarize some basic commands that are commonly used with Docker. These are especially helpful for those managing a Docker installation from a CLI (as opposed to using Portianer or other products, such as Synology, which integrate Docker and provide a GUI for management).

