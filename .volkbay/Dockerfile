FROM osrf/ros:melodic-desktop-full

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
	git wget autoconf automake nano m4 \
    libtool liblapack-dev libblas-dev \
    python3-catkin-tools python3-osrf-pycommon \
    python3-vcstools python3-vcstool \
    x11-apps
	
ENV WORKSPACE /uslam_ws
ENV ROS_VER melodic

RUN mkdir -p $WORKSPACE/src && \
	cd $WORKSPACE && \
	catkin init && \
	catkin config --extend /opt/ros/${ROS_VER} && \
	catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release

RUN cd ${WORKSPACE}/src && \
    git clone https://github.com/uzh-rpg/rpg_ultimate_slam_open.git
    
RUN cd ${WORKSPACE}/src && \
    sed -i 's+git@github.com:+https://github.com/+g' rpg_ultimate_slam_open/dependencies.yaml && \
    vcs-import < rpg_ultimate_slam_open/dependencies.yaml && \
    catkin build ze_vio_ceres -j$(nproc)

RUN	echo "export NVIDIA_VISIBLE_DEVICES=all" >> /root/.bashrc
RUN	echo "export NVIDIA_DRIVER_CAPABILITIES=all" >> /root/.bashrc
RUN	echo "export QT_X11_NO_MITSHM=1" >> /root/.bashrc

ENTRYPOINT ["/bin/bash"]
