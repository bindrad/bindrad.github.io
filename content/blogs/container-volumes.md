---
title: "From Ephemeral to Everlasting: Understanding Container Volumes"
date: 2024-01-30T00:00:00-07:00
draft: false
github_link: "https://github.com/bindrad/bindrad.github.io"
author: "Dhruv Bindra"
tags:
  - Containers
  - Docker
  - Volumes
image: /images/volumes.gif
description: ""
toc: 
---

Containers are small, lightweight bundles containing one or more applications and their dependencies, essential for running code. The beauty of containers lies in their ability to package all necessary components, making applications highly portable.


## Why does a Container Need Persistent Storage?

Containers are ephemeral, necessitating a means to store persistent data. While containers carry code, runtime, and dependencies wherever they go, they don't bring along application-generated data to maintain a lightweight footprint. Real-world applications, especially databases like Postgres, require data persistence. If a Postgres container generates data, it's crucial to retain that data even if the container is moved or restarted. This is where persistent storage becomes essential.

## How to Attach a Volume to a Container?

One effective method is by utilizing Docker volumes. Volumes are specifically designed to support stateful applications, ensuring that data is stored permanently in a persistent storage solution. 

Let's demonstrate this by mounting a volume to a container and verifying the content of same volume in another container:

Run a alpine container and mount volume called <mark>demo</mark> to it.
```
$ docker run -it -v demo:/demo alpine
```

Check the file system, you can see <mark>demo</mark> directory.
```
/ # ls
bin    demo   dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
```

Let's change the directory and create a file inside it with some content.
```
/ # cd demo/
/demo # echo "Hello World!" > demo.txt
/demo # cat demo.txt
Hello World!
```

Exit from alpine container, container will stop running as there is no process running inside it.
```
/demo # exit
```

Now, let's create another container using Fedora Docker image. Run the fedora docker image.
```
$ docker run -it -v demo:/lets_verify fedora
```

Check the file system, you can see <mark>lets_verify</mark> directory.
```
# ls
afs  bin  boot  dev  etc  home  lets_verify  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Change the directory to <mark>lets_verify</mark> and check the content of it. We can see the exact same content that we wrote in alpine container. This is the beauty of volumes in container, we can persist data while using containers.
```
# cd lets_verify/
# ls
demo.txt
# cat demo.txt
Hello World!
```

Exit from the container
```
[root@66ecfcdb8397 lets_verify]# exit
exit
```

Additionally, you can manually create volumes and configure permissions. For more details, consult the [official Docker documentation](https://docs.docker.com/storage/volumes/).
