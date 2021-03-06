---
title: Automated Build
permalink: /extend/docker/tutorial/automated-build/
---

* TOC
{:toc}

An important part of the Docker ecosystem is a *Docker registry*. A Docker registry acts as a folder of images; 
it takes care of storing and building images as well. 
[Docker Hub](https://hub.docker.com/) is the official Docker registry. 
There are also alternative registries, such as [Quay](https://quay.io/), or completely private registries 
such as [AWS ECR](https://aws.amazon.com/ecr/) where we are keen to host your images for you.

We support public and private images on both Docker Hub and Quay registries, and private images on AWS ECR. 
Other registries are not yet supported. 

## Working with a Registry
In order to run an image, *pull* (`docker pull`) that image to your machine. The `docker run` 
command does that automatically for you. So, when you do:

    docker run -i quay.io/keboola/base-php70
    
you will see something like this:

    Unable to find image 'quay.io/keboola/base-php70:latest' locally
    latest: Pulling from keboola/base-php70 
    a3ed95caeb02: Pull complete

When you build an image locally, *push* (`docker push`) it to the Docker registry. Then the
image can be shared with anyone (if public) or with your organization (if private). 

Because the image is defined only by the Dockerfile instructions (and optionally *build context*), you can take 
a shortcut and give the registry only the Dockerfile. The registry will then build the image on its own
infrastructure. This is best done by setting up an automated build which links a git repository 
containing the Dockerfile (and usually the application code) with the registry. 

## Setting up a Repository on Quay
This may get slightly confusing because we will create a new *Image Repository* and link
that to an existing *Github Repository*. Use the 
[sample repository](https://github.com/keboola/docs-docker-example-basic) 
created in [tutorial](/extend/docker/tutorial/howto/).

Create an account and organization, and then *Create a New Repository*:

{: .image-popup}
![Create Repository](/extend/docker/tutorial/quay-intro.png)

In the repository configuration, select *Link to a Github Repository Push*: 

{: .image-popup}
![Repository configuration](/extend/docker/tutorial/quay-new-repository.png)

Then link the image repository to a Github repository
(you can use our [sample repository](https://github.com/keboola/docs-docker-example-basic)):

{: .image-popup} 
![Link repositories](/extend/docker/tutorial/quay-link-repository.png)

After that, configure the build trigger. The easiest way to do that is setting the trigger to `All Branches and Tags`. 
It will trigger an image rebuild on every commit to the repository. 
You can also set the build trigger only to a specific branch, for example, `heads/master`:

{: .image-popup}  
![Configure build trigger for branch](/extend/docker/tutorial/quay-build-trigger-master.png)

An alternative option is to configure the trigger to a specific tag. For Semantic versioning, 
the following regular expression `^tags/[0-9]+\.[0-9]+\.[0-9]+$` ensures the image is rebuilt only when you create a new tag.
 
{: .image-popup}
![Configure build trigger for tag](/extend/docker/tutorial/quay-build-trigger-tag.png)

Regardless of your chosen approach, finish setting up the trigger by completing the wizard:

{: .image-popup}
![Configure build trigger](/extend/docker/tutorial/quay-build-trigger.png)

Pushing a new commit into a git repository or creating a new tag (depending on the trigger setting) will now
trigger a new build of the Docker Image. Also note that the image will automatically inherit the git repository tag 
or branch name. So, when you push a commit to the `master` branch, you will get an image with a tag master (which will
move away from any older image builds). When creating a `1.0.0` tag, you will get an image with a `1.0.0` tag.

## Setting up a Repository on AWS ECR

Contact us at [support@keboola.com](mailto:support@keboola.com) to create a new AWS ECR registry and AWS push credentials.

Once you received your AWS ECR registry URI and AWS credentials from us, the manual push process is as follows:
 
Retrieve the Docker login command that you can use to authenticate your Docker client to your registry:
    
    $ aws ecr get-login --region us-east-1
    docker login -u AWS -p *** -e none https://147946154733.dkr.ecr.us-east-1.amazonaws.com    

Run the Docker login command that was returned in the previous step.

    $ docker login -u AWS -p *** -e none https://147946154733.dkr.ecr.us-east-1.amazonaws.com
    Login Succeeded    

Build your Docker image.
    
    $ docker build -t {registryId} .

After the build completes, tag your image so you can push the image to this repository.

    $ docker tag {registryId}:latest 147946154733.dkr.ecr.us-east-1.amazonaws.com/{registryId}:latest

Run the following command to push this image to your newly created AWS repository.

    $ docker push 147946154733.dkr.ecr.us-east-1.amazonaws.com/{registryId}:latest
    
You can follow similar steps to push a tagged image or to integrate this push in your CI pipeline.
