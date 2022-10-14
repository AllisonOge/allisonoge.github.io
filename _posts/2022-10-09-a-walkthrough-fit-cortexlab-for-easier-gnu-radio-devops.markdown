---
layout: post
comments: true
title:  "A walkthrough FIT/CorteXlab for easier GNU Radio devOps"
excerpt: Explore a new open-source application that eases GNU Radio development with docker
date:   2022-10-09 07:26:24 +0100
updated:   2022-10-14 10:51:24 +0100
categories: devOps
---
<figure style="text-align:center">
  <img src="/images/clb-for-devops.png" alt="docker pull console">
  <figcation>Dockerizing GNU Radio</figcation>
</figure>

If you have struggled to setup your GNU Radio environment for a software-defined radio project, then you are like me. Dealing with all those dependencies and compatibility issues with different operating systems is really a tough nut to crack. Well, FIT/CorteX lab to the rescue. What is FIT/CorteX lab? It is one of the  testbeds of the [Future Internet of Things (FIT) initiative][fit-main]{:target="_blank"}. There are a lot of use cases of CorteXlab which can be found in the [documentation][clb-docs]{:target="_blank"}, however I want to address the issue of reproducibilty. How easily can I have a GNU Radio environment up and ready for my development? This actually is the reason I encountered FIT/CorteXlab. I have always fancied the capability of docker but my limited knowledge did not allow me make my dream of dockerizing GNU Radio a reality. So, coming across an open source platform that completely fulfil my dreams was such a daze. Hence in this blog, I will walk you through how to use this free open source platform to your advantange and maybe get you excited about the other benefits of FIT/CorteXlab. 

Let's begin by heading off to the the toolchain build [repo][clb-toolchain]{:target="_blank"}. It has a bash script that builds GNU Radio and all its dependencies as well as some OOT modules. We are one step closer to easing up the development setup but where does docker come in? Great question. Obviously, the bash script was built for a particular operating system and if you have noticed the debian system (sorry Windows users). So, docker makes it possible to run containers within any operating system that supports docker which are Windows, MacOS and Linux (hey Windows guys I have gat you). These containers can run debian docker images, so, provided we have a dockerfile that runs this bash script in a debian docker image, we can have our GNU Radio application running in docker on our local machine. Therefore, let's go through the steps, are you ready?

First, install docker in your local machine. To do that, head off to [this site][docker-install]{:target="_blank"}. If you prefer, you can install the desktop version. So, now that docker is up and ready let's get the debian docker image that builds GNU Radio. If you have used GNU Radio for a while then you know there are two versions of the software with major changes from each other, gnuradio-3.7 and otherwise. CorteXlab docker images also takes that into consideration. The list of docker images with GNU Radio versions they support can be found [here][clb-images]{:target="_blank"}. In this blog, I will walk through the steps discussed using a Linux OS (Ubuntu 18.04 LTS). So, the same steps can be taken for other operating systems but would require some added research on your part. 

In a terminal, we would run the following command to pull the docker image of any choice. In this case, I chose the [`m1mbert/cxlb-gnuradio-3.10:1.1`][clb-hub]{:target="_blank"} image for gnuradio-3.10.

{% highlight shell %}
docker pull m1mbert/cxlb-gnuradio-3.10:1.1
{% endhighlight %}

<figure style="text-align:center">
  <img src="/images/docker-pull.png" alt="docker pull console">
  <figcation>Console for <em>docker pull m1mbert/cxlb-gnuradio-3.10:1.1</em> command</figcation>
</figure>
<figure style="text-align:center">
  <img src="/images/verify-successful-pull.png" alt="docker images console">
  <figcation>Verify successfull image pull with <em>docker images</em> command</figcation>
</figure>

Next, we run the image with the command:

{% highlight shell %}
docker run -dit --net=host --expose 2222 --privileged m1mbert/cxlb-gnuradio-3.10:1.1
{% endhighlight %}

<figure style="text-align:center">
  <img src="/images/run-docker-instance.png" alt="docker run console">
  <figcation>Console for <em>docker run -dit --net=host --expose 2222 --privileged m1mbert/cxlb-gnuradio-3.10:1.1</em> command</figcation>
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

**Next Steps**

There are a few heads up you should be aware of, so I would bring you up to speed in that regard. You might wonder how would one run GUI applications in GNU Radio? That is truely a genuine concern. Recall earlier that I mentioned that the CorteXlab build toolchain (bash script) installs some OOT modules. Well, now is the time to discuss them. An important module which addresses the ability to run GUI applications is called the [Bokeh GUI][gr-bokeh]{:target="_blank"}. It is written by [Kartik Patel][kartik1995]{:target="_blank"}. The module allows a user to access the GUI widgets and sinks in a remote computer through a network. If you are thinking we will access the GUI plot of our running GNU radio program through the browser then you have guessed it right	&#128521;. As promised, I will show an example of Bokeh GUI in action in this post.

For a test, the following flow graph should suffice. It is the simulation of a signal source using BokehGUI time sink and waterfall plot.
<figure style="text-align:center">
  <img src="/images/grc-flowgraph.png" alt="bokehgui test">
  <figcation>Simple simulation flow graph to demonstrate BokehGUI in action</figcation>
</figure>

In the docker container, run the following commands to clone the repository and run the flow graph.

```shell
 git clone https://github.com/AllisonOge/bokehgui-test.git
 cd bokehgui-test/
 gnuradio-companion test_bokehgui.grc
```

At this point, you should see the flow graph and be able to execute it. However, there is just one more step if we want to access the port through the browser on our local machine. We need to forward the port in the container to that of our local machine. So, close the ssh by exiting the command line with `exit` command in the console or a combination of the keys `CTRL`+`D` in a Windows machine (macOS guys would have to figure out this one).

Now run the following command to make that connection (called [port forwarding](https://phoenixnap.com/kb/ssh-port-forwarding){:target="_blank"}) with the container. It is an adjustment to the previous ssh command I have shown.

```shell
ssh -Xp 2222 -L localhost:5006:localhost:5006 root@localhost
```

<figure style="text-align:center">
  <img src="/images/bridging-ports-for-bokehgui.png" alt="port forwarding">
  <figcation>Port forwarding in ssh command to allow network access</figcation>
</figure>

Now run the flow graph again and access the port shown in the console of the gnuradio program and voila! Your result should look so much like this.

<figure style="text-align:center">
  <img src="/images/bokeh-gui-browser.png" alt="bokehgui in action">
  <figcation>Bokeh GUI in action at port 5007</figcation>
</figure>

This brings us to the end of this post. I hope it was insightful. Do engage with the post and share your thoughts in the comment section and I would be glad to respond. This will sound as a strong encouragement to make other relevant posts. Have a wonderful rest of the day! Au revior &#128075;.

[fit-main]: http://www.cortexlab.fr/
[clb-docs]: https://wiki.cortexlab.fr/doku.php
[clb-toolchain]: https://github.com/CorteXlab/cxlb-build-toolchain
[docker-install]: https://docs.docker.com/engine/install/
[clb-images]: https://wiki.cortexlab.fr/doku.php?id=docker_images
[clb-hub]: https://hub.docker.com/r/m1mbert/cxlb-gnuradio-3.10
[gr-bokeh]: https://github.com/gnuradio/gr-bokehgui
[kartik1995]: https://www.linkedin.com/in/golappagouda-patil/
