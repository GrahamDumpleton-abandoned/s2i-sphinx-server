# Sphinx Site Generator

This repository implements a [Sphinx](http://www.sphinx-doc.org) site generator and hosting service. It is implemented as a [Source-to-Image](https://github.com/openshift/source-to-image) (S2I) builder and can be used to generate a standalone Docker-formatted image which you can run with any container runtime supporting Docker-formatted images. Alternatively the S2I builder can be used in conjunction with OpenShift to handle both the build and deployment of the generated Sphinx formatted site.

## Integration With OpenShift

To use this with OpenShift, it is a simple matter of creating a new application within OpenShift, pointing the S2I builder at the Git repository containing your Sphinx document source.

As an example, to build and host the the [Sphinx site](https://github.com/sphinx-doc/sphinx), you only need run:

```
oc new-build getwarped/s2i-sphinx-server:1.4~https://github.com/sphinx-doc/sphinx.git --name sphinx-site --env DOCUMENT_ROOT=doc

oc new-app sphinx-site

oc expose svc/sphinx-site
```

It was necessary in this case to create a build prior to the application as we needed to set the ``DOCUMENT_ROOT`` environment variable for the build. If this wasn't necessary then could have used just the first command but replace ``new-build`` with ``new-app``.

To have any changes to your document source automatically redeployed when changes are pushed back up to your Git repository, you can use the [web hooks integration](https://docs.openshift.com/container-platform/latest/dev_guide/builds.html#webhook-triggers) of OpenShift to create a link from your Git repository hosting service back to OpenShift.

To make it easier to deploy a Sphinx site, a template for OpenShift is also included. This can be loaded into your project using:

```
oc create -f https://raw.githubusercontent.com/getwarped/s2i-sphinx-server/master/template.json
```

Once loaded, select the ``sphinx-server`` template from the web console when wanting to add a new site to the project.

The template details are:

```
Name:		sphinx-server
Description:	Sphinx Server
Annotations:	tags=instant-app,sphinx

Parameters:
    Name:		APPLICATION_NAME
    Description:	The name of the application.
    Required:		true
    Value:		sphinx-server

    Name:		SOURCE_REPOSITORY
    Description:	Git repository for source.
    Required:		true
    Value:		<none>
    
    Name:		SOURCE_DIRECTORY
    Description:	Sub-directory of repository for source files.
    Required:		false
    Value:		<none>
    Name:		DOCUMENT_ROOT
    
    Description:	Sub-directory of source directory for documents.
    Required:		false
    Value:		<none>
    
    Name:		BUILD_MEMORY_LIMIT
    Description:	Build time memory limit.
    Required:		true
    Value:		512Mi

Objects:
    ImageStream		${APPLICATION_NAME}
    ImageStream		${APPLICATION_NAME}-s2i
    DeploymentConfig	${APPLICATION_NAME}
    Service		${APPLICATION_NAME}
    Route		${APPLICATION_NAME}
```

The ``APPLICATION_NAME`` and ``SOURCE_REPOSITORY`` must be specified. Override the ``BUILD_MEMORY_LIMIT`` and set it to a higher memory value if the ``sphinx`` application keeps getting killed when processing the site data.

## Standalone Docker Images

To create a standalone Docker-formatted image, you need to [install](https://github.com/openshift/source-to-image/releases) the ``s2i`` program from the Source-to-Image (S2I) project locally. Once you have this installed, you would run within your Git repository:

```
s2i build . getwarped/s2i-sphinx-server:1.4 mysphinxsite
```

In this case this will create a Docker-formatted image called ``mysphinxsite``. You can then run the image using:

```
docker run --rm -p 8080:8080 mysphinxsite
```