---
layout: default
title: Shipwright | The right way to build, tag and ship shared Docker containers.
---

<strong>Shipwright | The right way to build, tag and 
ship Docker containers.</strong>


Shipwright builds shared Docker images within a git repository
in **the right order** and publishes them tagged with git's revision/branch
information so  you'll never loose track of an image's origin.

It's the perfect tool for building and publishing your images to places
like Docker Hub or your own private registry. Have a look at  [our motivation](docs/motivation.md) to see why we built it and the pain points it solves for you.


Installation
============

Shipwright is a simple python script you can install with pip

	$ pip install shipwright



Quickstart
==========

Once installed, simply change to a project of yours that contains multiple Dockerfiles and is in git. 

Add a json formated file named `.shipwright.json` to the root directory 
of your project. At minimum it should contain
the version number of your Shipwright config  and a `namespace` which
is either your docker hub user name or the URL to your private repository.

1.0 is the current version for the config.

```json
{
  "version": 1.0,
  "namespace": "[your docker hub name or private repository]"
}
```

Additionally your config file can map directory names to alternative
docker repositories. For example here is a `.shipwright.json`
for the docker hub user `shipwright` that also maps the root of the 
git repository to the docker image `shipwright/shared` and the `/foo`
directory to `shipwright/awesome_sauce`.

```json
{
  "version": 1.0,

  "namespace": "shipwright",
  "names": {
    "/": "shipwright/shared",
    "/foo": "shipwright/awesome_sauce"
}
```

Now you can build all the docker images in the git repo by simply changing
to any directory under your git repo and runnig:

	$ shipwright
	
This will recurse through all the directories, looking for ones that contain a Dockerfile. Shipwright will build these dockefiles in order and by default tag them with `<namespace>/<dirname>:<git commit>` along with `<namespace>/<dirname>:<git branch>` and `<namespace>/<dirname>:latest`

Working example
---------------

We have [a sample shipwright project](https://github.com/6si/shipwright-sample) you can use if you want to try this out right away.

```bash
$ git clone https://github.com/6si/shipwright-sample.git
$ cd shipwright-sample
$ shipwright 
```

**NOTE: you can use any username you'd like while building locally. In the above example we use `shipwright`. Nothing is published unless you use the `push` command. For your own projects  substitue `shipwright` in the above example with you (or your organizations) official docker hub username or private respository.**

Notice that if you run the `shipwright` a second time it will return immediatly without doing anything. Shipwright is smart enough to know nothing has changed.

Shipwright really shines when you switch git branches.

```bash
$ git checkout new_feature
$ shipwright
```


Notice that shipwright only rebuilt  the shared library and service1 ignoring the other projects because they have a common git ancestory. Running `docker images` however shows that all the images in the git repository have been tagged with the latest git revision, branch and `latest`. 

In fact as Shipwright builds  containers it rewrites the Dockerfiles so that they require the base images with tags from the current git revision. This ensures that the entire build is deterministic and reproducable.


Building
=========


By default, if you run shipwright with no arguments, it will build all Dockerfiles
in your git repo. You can specify one or more `specifiers` to select fewer images to build. For example you can build a single images and it's dependencies by simply
specifiying it's name on the command line

```
$ shipwright <namespace>/some_image
```

Run `shipwright --help' for more examples of specifiers and their uses.


Publishing
==========

With one command Shipwright can build your images and push them to a remote repository.

```
$ shipwright push
``` 

If you like you can just push your latest images without building.


```
$ shipwright push --no-build 
```

The same specifiers for building also work with `push`. Yop might use this
to build an entire tree in one step then push a spcefic image like so.

```
$ shipwright build
$ shipwright push -e <namespace>/public_image
```

Purging
=======

After using Shipwright for a bit, images tend to build up that you no longer
use. Shipwright has a experimental command `purge` that will remove all stale 
images.  A stale image is an image that is not the latest of at 
least one branch. Images tagged with branch names that are no longer in git
will also be removed.

```
$ shipwright purge
```
