---
layout: post
comments: true
title:  "Fit Cortex Lab with GNU Radio using Docker: A Comprehensive Tutorial (2)"
excerpt: ""
date:   2023-10-10 00:11:24 +0100
categories: devOps
---
In this part of the tutorial, we'll delve deeper into Fit Cortex Lab, covering topics such as reserving nodes, submitting tasks, and running experiments on the actual hardware. We'll also explore advanced use cases and techniques for harnessing the power of Fit Cortex Lab for your SDR experiments. It is best to look at the first part of this tutorial to get the best of it.

# Preparing Your Docker Image for Fit Cortex Lab

In this section, we'll prepare our Docker image for use in Fit Cortex Lab. Follow these steps carefully to ensure your image is ready for experimentation:

**Step 1: Docker Commit**
- Begin by closing GNU Radio Companion (GRC) if it's open, as we won't need it anymore.
- In your terminal, run the following command to list all running Docker containers and make note of the container ID:
  ```bash
  docker ps
  ```
- You'll need the container ID for the next steps.
- Now, let's create a new Docker image from this container. Use the `docker commit` command, specifying a message and your Docker repository and tag:
  ```bash
  docker commit -m "Preparation for FIT/CorteXlab" CONTAINER_ID YOUR_DOCKER_REPO:TAG
  ```
  Replace `CONTAINER_ID` with the actual container ID you noted earlier and set `YOUR_DOCKER_REPO:TAG` with your desired repository and tag name.

**Step 2: Docker Tag**
- After committing changes, we need to tag the image before pushing it to the Docker registry. Execute the following command:
  ```bash
  docker tag YOUR_DOCKER_REPO:TAG YOUR_DOCKER_REPO:TAG
  ```
  This ensures that the image is correctly tagged.

**Step 3: Docker Push**
- Finally, it's time to push the Docker image to the Docker Hub or your desired registry. Run this command:
  ```bash
  docker push YOUR_DOCKER_REPO:TAG
  ```
  Allow some time for the image to be uploaded to the registry.

**Note**: The above method provides a quick way to prepare a Docker image for Fit Cortex Lab. However, it doesn't support automation as there's no accompanying Dockerfile. For a more proper and automated approach, it's recommended to create a Dockerfile with your desired setup. Here's an example of a Dockerfile structure:

```Dockerfile
# Use an existing image as the base
FROM lcardoso/cxlb-gnu-radio-3.8:1.1

# Add commands to customize your image
RUN git clone <your_git_repository>
COPY <your_files_to_copy> /

# Set entry point or other configurations
ENTRYPOINT ["/your_entry_script.sh"]
```

By creating a Dockerfile, you can fully automate the setup of your Docker image, including adding dependencies, copying files, and defining entry points.

# Connecting to FIT Cortex Lab

Now that your Docker image is ready, let's proceed to connect to FIT Cortex Lab. But before we delve into the connection process, let's briefly review the FIT Cortex Lab experimental tools:

**OAR (Resource Allocation):**
- OAR allows you to make reservations on the testbed. Reservations can be exclusive or shared, with each having an identification number. Key commands include `oarsub` for submission and `oardel` for deletion. Reservations are time-limited, and exceeding the reservation time will result in automatic termination.

**MINUS (Experiment Management):**
- MINUS handles experiment creation, submission, and management. An experiment consists of code, libraries, and a scenario description. For Docker-based experiments, the code resides within the Docker image. Tasks within experiments are identified by ID numbers and executed based on submission order.

**DrawGantt:**
- DrawGantt is a tool to visualize testbed reservations, showing reserved and free slots. It helps users find available time slots for their experiments.

Now, let's move on to the next interactive session.

# Connecting to FIT Cortex Lab

In this session, we'll connect to FIT Cortex Lab, create a task folder, scenario, and task file, reserve a testbed, and verify the reservation status.

**Step 1: Connect to Airlock**
- Open your terminal and connect to Airlock using SSH. Replace `USERNAME` with your username and `NODE` with the chosen gateway node (e.g., `gw.cortexlab.fr`).
  ```bash
  ssh -Y USERNAME@NODE
  ```

**Step 2: Create Task Folder and Scenario**
- In your Airlock terminal, navigate to the tasks folder or create it if it doesn't exist:
  ```bash
  cd ~/tasks
  mkdir cortexlab-tutorial
  cd cortexlab-tutorial
  ```
- Use a text editor like `pico` to create a `scenario.yaml` file:
  ```bash
  pico scenario.yaml
  ```
- Inside `scenario.yaml`, specify the scenario details:
  ```yaml
  description: "OFDM RX Cortex Lab Tutorial"
  duration: 600  # 10 minutes
  nodes:
    nodeXX:
      container:
        image: YOUR_DOCKER_REPO:TAG
        command: "/usr/sbin/sshd -p 2222 -D"
  ```
  - Replace `YOUR_DOCKER_REPO:TAG` with your Docker image repository and tag.
  - Replace `nodeXX` with your assigned node.
- Save and exit the text editor (`pico`).

**Step 3: Create Task**
- Use the following command to create the task:
  ```bash
  minus task create cortexlab_tutorial -s scenario.yaml
  ```
- Make note of the task ID provided.

**Step 4: Reserve the Testbed**
- To reserve the testbed, run the following command:
  ```bash
  oarsub -t "inner=YOUR_CONTAINER_RESERVATION_NUMBER" -l "network_address in ('mnodeXX.cortexlab.fr')/nodes=1,walltime=0:30:00" -I
  ```
  - Replace `YOUR_CONTAINER_RESERVATION_NUMBER` with the container reservation number.
  - Replace `mnodeXX` with your assigned node.
  - Adjust the walltime as needed.

**Step 5: Verify Reservation Status**
- Check the status of your reservation by running:
  ```bash
  minus testbed status
  ```
- Ensure your reservation is running.

You've now connected to FIT Cortex Lab, created a task, reserved a testbed, and verified your reservation. In the next interactive session, we'll launch the experiment and test your OFDM receiver.
