---
layout: post
comments: true
title:  "A walkthrough FIT/CorteXlab for easier GNU Radio devOps"
excerpt: "Explore a new open-source application that eases GNU Radio development with docker"
date:   2022-10-09 07:26:24 +0100
updated:   2022-11-15 11:47:24 +0100
categories: devOps
---
<figure style="text-align:center">
  <img src="/images/clb-for-devops.png" alt="devops with gnuradio and docker">
  <figcation>Dockerizing GNU Radio</figcation>
</figure>

If you have struggled to setup your GNU Radio environment for a software-defined radio project, then you are like me. Dealing with all those dependencies and compatibility issues with different operating systems is really a tough nut to crack. Well, FIT/CorteX lab to the rescue. What is FIT/CorteX lab? It is one of the  testbeds of the [Future Internet of Things (FIT) initiative][fit-main]{:target="_blank"}. There are a lot of use cases of CorteXlab which can be found in the [documentation][clb-docs]{:target="_blank"}, however I want to address the issue of reproducibilty. How easily can I have a GNU Radio environment up and ready for my development? And how easily can I return to a past project or reproduce the results of a project on my local machine? This actually is the reason I encountered FIT/CorteXlab. I have always fancied the capability of docker but my limited knowledge did not allow me make my dream of dockerizing GNU Radio a reality. So, coming across an open source platform that completely fulfil my dreams was such a daze. Hence in this blog, I will walk you through how to use this free open source platform to your advantange and maybe get you excited about the other benefits of FIT/CorteXlab. 

Let's begin by heading off to the the toolchain build [repo][clb-toolchain]{:target="_blank"}. It has a bash script that builds GNU Radio and all its dependencies as well as some OOT modules. We are one step closer to easing up the development setup but where does docker come in? Great question. Obviously, the bash script was built for a particular operating system and if you have noticed the debian system (sorry Windows users). So, docker makes it possible to run containers within any operating system that supports docker which are Windows, MacOS and Linux (hey Windows guys I have gat you). These containers can run debian docker images, so, provided we have a dockerfile that runs this bash script in a debian docker image, we can have our GNU Radio application running in docker on our local machine. Therefore, let's go through the steps, are you ready?

First, install docker in your local machine. To do that, head off to [this site][docker-install]{:target="_blank"}. If you prefer, you can install the desktop version. So, now that docker is up and ready let's get the debian docker image that builds GNU Radio. GNU Radio has a few common major versions and fortunately CorteXlab docker images also takes that into consideration. The list of docker images with GNU Radio versions they support can be found [here][clb-images]{:target="_blank"}. In this blog, I will walk through the steps discussed using a Linux OS (Ubuntu 18.04 LTS). So, the same steps can be taken for other operating systems but would require some added research on your part. 

In a terminal, we would run the following command to pull the docker image of any choice. In this case, I chose the [`m1mbert/cxlb-gnuradio-3.10:1.2`][clb-hub]{:target="_blank"} image for gnuradio-3.10.

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

Next, we run the image with the command:

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


The command runs an interactive container that is open to the host network at port 2222. With the container running (you can verify with this command `docker ps`), we can connect to the container with ssh using the command:

{% highlight shell %}
ssh -Xp 2222 root@localhost
{% endhighlight %}

If all goes as planned then you should be in the running container with gnuradio-3.10 installed. To verify that GNU Radio works run the command `gnuradio-companion`. And with that we conclude the development setup with FIT/CorteXlab. 

This brings us to the end of this post. I hope it was insightful. Do engage with the post and share your thoughts in the comment section and I would be glad to respond. This will sound as a strong encouragement to make other relevant posts. Have a wonderful rest of the day! Au revior &#128075;.


<!-- **Next Steps**

Now what? A little dive into the world of Fit/Cortex Lab.  -->


[fit-main]: http://www.cortexlab.fr/
[clb-docs]: https://wiki.cortexlab.fr/doku.php
[clb-toolchain]: https://github.com/CorteXlab/cxlb-build-toolchain
[docker-install]: https://docs.docker.com/engine/install/
[clb-images]: https://wiki.cortexlab.fr/doku.php?id=docker_images
[clb-hub]: https://hub.docker.com/r/m1mbert/cxlb-gnuradio-3.10
[gr-bokeh]: https://github.com/gnuradio/gr-bokehgui
[kartik1995]: https://www.linkedin.com/in/golappagouda-patil/
