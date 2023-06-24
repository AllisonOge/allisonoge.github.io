---
layout: post
comments: true
title:  "Dockering GNU Radio for Easy Development Setup"
excerpt: "Explore a new open-source application that eases GNU Radio development with docker"
date:   2022-10-09 07:26:24 +0100
updated:   2023-06-24 05:11:24 +0100
categories: devOps
---
<figure style="text-align:center">
  <img src="/images/clb-for-devops.png" alt="devops with gnuradio and docker">
  <figcation>Dockerizing GNU Radio</figcation>
</figure>

Setting up a GNU Radio environment for a software-defined radio project can be a challenging task, with various dependencies and compatibility issues across different operating systems. However, there's a solution to simplify this process with FIT/CorteX lab. In this blog post, we'll explore FIT/CorteX lab, an open-source platform that addresses the issues of reproducibility and provides an easy way to have a GNU Radio environment up and running for development. By leveraging Docker, we can achieve the dream of dockerizing GNU Radio and enjoy the additional benefits of FIT/CorteXlab. So, let's dive in and learn how to leverage this free and powerful platform.

## The Power of FIT/CorteXlab
[FIT/CorteX lab][fit-main]{:target="_blank"} is an essential testbed of the Future Internet of Things (FIT) initiative. It offers numerous use cases, which you can explore in their [comprehensive documentation][clb-docs]{:target="_blank"}. However, in this blog post, we'll focus on the aspect of reproducibility and how FIT/CorteXlab simplifies the setup of a GNU Radio environment, enabling easy project replication and returning to past projects on your local machine.

## Getting Started: Docker and GNU Radio
To begin, you'll need to install Docker on your local machine. Visit the [Docker website][docker-install]{:target="_blank"} and follow the installation instructions. Docker is supported on Windows, macOS, and Linux, making it versatile for various operating systems.

GNU Radio has different major versions, and FIT/CorteXlab takes this into consideration. You can find a list of Docker images available with support for different GNU Radio versions on the FIT/CorteXlab [website][clb-images]{:target="_blank"}. In this blog post, we'll focus on the Linux OS (Ubuntu 18.04 LTS), but the general steps can be adapted for other operating systems with additional research.

## Step 1: Pulling the Docer Image
Open a terminal and execute the following command to pull the Docker image of your choice. For example, we'll use the [`m1mbert/cxlb-gnuradio-3.10:1.2`][clb-hub]{:target="_blank"} image for GNU Radio version 3.10.

{% highlight shell %}
docker pull m1mbert/cxlb-gnuradio-3.10:1.2
{% endhighlight %}

<figure style="text-align:center">
  <img src="/images/docker-pull.png" alt="docker pull console">
  <figcation>Console for <em>docker pull m1mbert/cxlb-gnuradio-3.10:1.2</em> command</figcation>
</figure>
<figure style="text-align:center">
  <img src="/images/verify-successful-pull.png" alt="docker images console">
  <figcation>Verify successfull image pull with <em>docker images</em> command</figcation>
</figure>

This command downloads the Docker image to your local machine, ensuring you have the necessary environment to run GNU Radio.

## Step 2: Running the Docker Image
Once the image is pulled, we can run it using the following command:

{% highlight shell %}
docker run -dit --net=host --expose 2222 --privileged m1mbert/cxlb-gnuradio-3.10:1.2
{% endhighlight %}

<figure style="text-align:center">
  <img src="/images/run-docker-instance.png" alt="docker run console">
  <figcation>Console for <em>docker run -dit --net=host --expose 2222 --privileged m1mbert/cxlb-gnuradio-3.10:1.2</em> command</figcation>
</figure>
<figure style="text-align:center">
  <img src="/images/verify-successful-run.png" alt="docker ps console">
  <figcation>Verify successful docker instance run with <em>docker ps</em> command</figcation>
</figure>

This command launches an interactive container that is connected to the host network and exposes port 2222. Verify that the container is running by executing the `docker ps` command.

## Step 3: Connecting to the Container
To connect to the running container, use the SSH command with the following parameters:

{% highlight shell %}
ssh -Xp 2222 root@localhost
{% endhighlight %}

Assuming everything went smoothly, you should now be inside the running container with GNU Radio version 3.10 installed. You can verify the installation by running the `gnuradio-companion` command.

## Conclusion

Congratulations! You have successfully set up a development environment with FIT/CorteXlab and Dockerized GNU Radio. This powerful combination simplifies the GNU Radio setup process and ensures reproducibility of your projects. By leveraging FIT/CorteXlab, you can easily return to past projects and share your insights with the community.

I hope this blog post has been insightful and valuable to you. Feel free to engage with the post in the comment section below. Au revior &#128075;. 

[fit-main]: http://www.cortexlab.fr/
[clb-docs]: https://wiki.cortexlab.fr/doku.php
[clb-toolchain]: https://github.com/CorteXlab/cxlb-build-toolchain
[docker-install]: https://docs.docker.com/engine/install/
[clb-images]: https://wiki.cortexlab.fr/doku.php?id=docker_images
[clb-hub]: https://hub.docker.com/r/m1mbert/cxlb-gnuradio-3.10
[gr-bokeh]: https://github.com/gnuradio/gr-bokehgui
[kartik1995]: https://www.linkedin.com/in/golappagouda-patil/
