# Docker Networking

Docker runs applications inside of containers, and applications need to communicate over lots of different networks. This means Docker needs strong networking capabilities.

Fortunately, Docker has solutions for container-to-container networks, as well as connecting to existing networks and VLANs. The latter is important for containerized apps that interact with functions and services on external systems such as VM’s and physical servers.

Docker networking is based on an open-source pluggable architecture called the Container Network Model(CNM). libnetwork is Docker’s real-world implementation of the CNM, and it provides all of Docker’s core networking capabilities. Drivers plug in to libnetwork to provide specific network topologies.

To create a smooth out-of-the-box experience, Docker ships with a set of native drivers that deal with the most common networking requirements. These include single-host bridge networks, multi-host overlays, and options for plugging into existing VLANs. Ecosystem partners can extend things further by providing their own drivers.

Last but not least, libnetwork provides a native service discovery and basic container load balancing solution.