---
layout: post
comments: true
title:  "CogNet: Demonstration of cognitive radio ad-hoc networks with FIT/CorteX lab"
excerpt: "How smart is my channel selection algorithm? Wanna find out"
date:   2022-12-23 07:26:24 +0100
# updated:   2022-10-14 10:51:24 +0100
categories: cognet
---

<!-- The Cortexlab toolchain builds some OOT modules in addition to GNU Radio. One important module that allows remote access to GUI applications is the [Bokeh GUI][gr-bokeh]{:target="_blank"}, written by [Kartik Patel][kartik1995]{:target="_blank"}. The module allows a user to access the GUI widgets and sinks in a remote computer through a network. If you are thinking we will access the GUI plot of our running GNU radio program through the browser then you have guessed it right	&#128521;. As promised, I will show an example of Bokeh GUI in action in this post.

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
</figure> -->