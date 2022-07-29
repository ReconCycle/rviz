# ROS RVIZ in Docker

RVIZ can be difficult with nvidia. If you have an Nvidia GPU you must follow the installation guide with Nvidia.

**VERY IMPORTANT** Run `xhost +` on the client in order to see the GUI.

## Installation with Nvidia

1. Install CUDA toolkit 11.3 as outlined in [ros-vision-pipeline](https://github.com/ReconCycle/ros-vision-pipeline).
2. `cp docker-compose.example.yml docker-compose.yml`
3. `docker-compose build --build-arg 'TARGET=gpu'`
4. `docker-compose up -d`

## Installation without Nvidia

1. `cp docker-compose.example.yml docker-compose.yml`
2. `docker-compose build --build-arg 'TARGET=cpu'`
3. `docker-compose up -d`
