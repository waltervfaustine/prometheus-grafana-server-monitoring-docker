# Enhancing Server Performance and Reliability: A Guide to Installing and Using Prometheus and Grafana for Server Monitoring using Docker


Monitoring your server infrastructure is crucial for maintaining the health and performance of your applications. Grafana and Prometheus are two powerful tools that can help you achieve this. In this article, we will walk you through the process of setting up Grafana and Prometheus using Docker. By the end of this guide, you will have a fully functional monitoring setup.
Project Folder Structure
Before we dive into the setup process, let's look at the folder structure you will need. Each file and folder has a specific role in the setup:

prometheus-grafana-server-monitoring-docker/
├── grafana/
│ └── grafana.ini
├── prometheus/
│ └── prometheus.yml
├── .env
├── build.sh
├── docker-compose.yml
├── LICENSE
├── README.md
└── .gitignore

Explanation of Folder Structure

1. grafana/
This directory contains configuration files for Grafana.
grafana.ini: This is the main configuration file for Grafana. It allows you to customize various settings such as security, data sources, and logging.

# Sample Grafana Configuration File

# [server] section: Configuration related to the Grafana server
[server]
http_port = 3000  # The port Grafana will listen on. Default is 3000.

# [security] section: Configuration related to Grafana security settings
[security]
admin_user = admin  # The username for the Grafana admin user. Default is 'admin'.
admin_password = ${GF_SECURITY_ADMIN_PASSWORD}  # The password for the Grafana admin user. Uses an environment variable for security.

# DOCUMENTATION:
# 1. **[server] section**: Contains settings related to the Grafana server.
#    - `http_port`: The port on which Grafana will be accessible. The default is 3000.
#
# 2. **[security] section**: Contains settings related to the security and authentication of Grafana.
#    - `admin_user`: Specifies the username for the Grafana admin user. By default, it is set to 'admin'.
#    - `admin_password`: Specifies the password for the Grafana admin user. It uses an environment variable (${GF_SECURITY_ADMIN_PASSWORD}) to avoid hardcoding sensitive information in the configuration file.
#
# 3. **Environment Variable**:
#    - `admin_password` uses the environment variable `${GF_SECURITY_ADMIN_PASSWORD}`. Ensure this environment variable is set in the Docker Compose file or in an `.env` file to provide the admin password securely.
#
# 4. **File Location**: This configuration file should be placed in the `./grafana` directory if you are using the Docker Compose configuration provided, so it maps correctly to `/etc/grafana` inside the container.

2. prometheus/
This directory holds the configuration files for Prometheus.
prometheus.yml: This file contains the configuration for Prometheus, including global settings and scrape configurations that define the metrics sources.

# Prometheus Configuration File

# Global configuration settings
global:
  scrape_interval: 15s  # How frequently to scrape targets by default

# Scrape configuration
scrape_configs:
  # Job for scraping Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']  # Prometheus server itself as a target

  # Scrape configuration for Node Exporter
  - job_name: 'node'  # A label to identify the scraping configuration for Node Exporter
    static_configs:
      - targets: ['cainam-node-exporter:9100']  # Target for Node Exporter metrics, typically the Node Exporter container

# DOCUMENTATION:
# 1. **global**: Global configuration parameters for Prometheus.
#    - `scrape_interval`: Specifies the default time interval between scrapes. Set to 15 seconds in this example.
# 2. **scrape_configs**: Defines the list of targets and how to scrape them.
#    - **job_name**: Identifies the job name for a group of targets.
#      - 'prometheus': A job to scrape the Prometheus server itself.
#      - **static_configs**: Lists static targets to be scraped.
#        - `targets`: A list of targets (hostnames and ports) to be scraped for each job.
#        - 'localhost:9090': The Prometheus server running on the local machine.
#      - 'my_app': A job to scrape a custom application.
#        - `targets`: Replace 'my_app_host:9100' with the actual hostname and port of your application.
# 3. **Adjustments**: Modify the `targets` section to include the actual endpoints you want Prometheus to monitor.
# 4. **File Location**: Ensure this configuration file is located at the path specified in the Docker Compose file (`./prometheus/prometheus.yml`).

3. .env
The .env file is used to define environment variables that are utilized in your Docker setup. These variables can be referenced in the Docker Compose file to manage configuration settings such as usernames and passwords securely.
# Environment Variables from .env file
#
COMPOSE_PROJECT_NAME=cainam
#   - Defines the project name for Docker Compose, which affects the names of containers and networks.
#
GF_SECURITY_ADMIN_USER=admin
#   - Sets the Grafana admin username to 'admin' for initial login.
#
GF_SECURITY_ADMIN_PASSWORD=INSERT_YOUR_DESIRED_STRONG_PASSWORD
#   - Sets the Grafana admin password using an environment variable from the .env file.
#   - Ensure to replace ${GF_SECURITY_ADMIN_PASSWORD} with the a
4. build.sh
This is a shell script used to build and manage your Docker Compose setup. It contains commands to bring up and tear down the Docker services.
#!/bin/bash
# Docker Compose Command for Deployment Management

# Move to the project root directory
cd ..

# Stop and remove all running containers defined in the Docker Compose file
docker-compose -f docker-compose-cainam.yml down

# Build images, recreate containers if necessary, and start all containers in detached mode
docker-compose -f docker-compose-cainam.yml up --build -d

# DOCUMENTATION:
# 1. **cd ..**: Moves up one directory to the project root where the `docker-compose.yml` file is located.
# 2. **docker-compose -f docker-compose.yml down**: Stops and removes all containers defined in the specified Docker Compose file.
# 3. **docker-compose -f docker-compose.yml up --build -d**: Builds the images if changes are detected, recreates containers if necessary, and starts all containers in detached mode.
# 4. **-f docker-compose.yml**: Specifies the path to the `docker-compose.yml` file.
# 5. **-d**: Detached mode - runs containers in the background.
# 6. **Usage**: Use this command to apply changes and start services after modifying the configuration or code.
5. docker-compose.yml
The docker-compose.yml file defines the Docker services for Grafana and Prometheus. It specifies the images to use, the ports to expose, and the volumes to mount for configuration files.
# Docker Compose Configuration for Prometheus and Grafana Monitoring Setup

# Specify the version of Docker Compose configuration format
version: "3.8"

# Define the services that will run in the Docker network
services:
  # Prometheus service configuration
  prometheus:
    # Use the latest version of Prometheus from Docker Hub
    image: prom/prometheus:latest
    # Specify the container name for easy identification
    container_name: cainam-prometheus
    # Map host machine's port 9090 to the container's port 9090 for Prometheus access
    ports:
      - "9090:9090"
    # Define volumes to persist Prometheus configuration
    volumes:
      - ./prometheus:/etc/prometheus # Volume for Prometheus configuration
    # Specify Prometheus command-line arguments
    command:
      - '--config.file=/etc/prometheus/prometheus.yml' # Use the Prometheus configuration file
    # Attach the service to the custom monitoring network
    networks:
      - cainam-network
    # Healthcheck configuration for Prometheus service
    healthcheck:
      test:
        [
          "CMD",
          "curl",
          "-f",
          "http://localhost:9090/-/healthy"
        ] # Health check endpoint
      interval: 30s # Check health status every 30 seconds
      timeout: 10s # Timeout duration for health check
      retries: 3 # Number of retries if health check fails

  # Grafana service configuration
  grafana:
    # Use the latest version of Grafana from Docker Hub
    image: grafana/grafana:latest
    # Specify the container name for easy identification
    container_name: cainam-grafana
    # Map host machine's port 3000 to the container's port 3000 for Grafana access
    ports:
      - "3000:3000"
    # Define volumes to persist Grafana configuration
    volumes:
      - ./grafana:/etc/grafana # Volume for Grafana configuration
    # Set environment variables for Grafana authentication and configuration
    environment:
      - GF_SECURITY_ADMIN_USER=admin # Admin user (can be set in .env file)
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD} # Admin password (set in .env file)
      - GF_PATHS_CONFIG=/etc/grafana/grafana.ini
    # Attach the service to the custom monitoring network
    networks:
      - cainam-network

    depends_on:
      - prometheus  # Depend on Prometheus service for proper startup order
      
    # Healthcheck configuration for Grafana service
    healthcheck:
      test:
        [
          "CMD",
          "curl",
          "-f",
          "http://localhost:3000/api/health"
        ] # Health check endpoint
      interval: 30s # Check health status every 30 seconds
      timeout: 10s # Timeout duration for health check
      retries: 3 # Number of retries if health check fails

  # Node Exporter service configuration
  node_exporter:
    # Use the Node Exporter Docker image
    image: prom/node-exporter
    # Specify a custom container name
    container_name: cainam-node-exporter
    # Map host machine's port 9100 to the container's port 9100
    ports:
      - "9100:9100"

    depends_on:
      - grafana # Depend on Grafana service for proper startup order
      - prometheus # Depend on Prometheus service for proper startup order

    # Attach the service to the monitoring network
    networks:
      - cainam-network

# Define networks for the services
networks:
  cainam-network:
    external: true # Use an external network named 'cainam-network'

# DOCUMENTATION:
# 1. This Docker Compose file uses version "3.8" for compatibility and features.
# 2. Configures Prometheus and Grafana services for monitoring and visualization.
# 3. Prometheus is accessed via port 9090 on the host machine.
# 4. Grafana is accessed via port 3000 on the host machine.
# 5. Volumes are defined for persisting Prometheus and Grafana configurations.
# 6. Environment variables are used for setting Grafana admin credentials.
# 7. Both services are attached to a custom bridge network named 'cainam-network'.
# 8. Health checks are configured to monitor the availability of Prometheus and Grafana.
# 9. Adjust settings and volumes based on specific deployment requirements.
# 10. Ensure to set the Grafana admin password securely using an environment variable in a .env file.
Step-by-Step Configuration
Step 1: Choose Your Configuration Approach
You have two approaches to set up Grafana and Prometheus:
Option 1: Prepare Configuration from Scratch
If you prefer to set up Grafana and Prometheus from scratch using the information provided in this guide:
Create a new directory for your project and navigate into it:

#!/bin/bash

# Create a new directory named `prometheus-grafana-server-monitoring-docker`
mkdir prometheus-grafana-server-monitoring-docker

# Move into the newly created directory
cd prometheus-grafana-server-monitoring-docker

# DOCUMENTATION:
# 1. **mkdir prometheus-grafana-server-monitoring-docker**: Creates a directory named `prometheus-grafana-server-monitoring-docker` in the current location.
# 2. **cd prometheus-grafana-server-monitoring-docker**: Changes the current directory to `prometheus-grafana-server-monitoring-docker`, allowing you to work within this directory.
# 3. **Usage**: Use these commands to set up a new directory for managing Prometheus and Grafana server monitoring configurations or scripts.
2. Set up the folder structure and configuration files as outlined below:
prometheus-grafana-server-monitoring-docker/
├── grafana/
│   └── grafana.ini
#   - Configuration file for Grafana, defining settings such as database, server, and security configurations.
│
├── prometheus/
│   └── prometheus.yml
#   - Configuration file for Prometheus, specifying scrape configurations, alerting rules, and other settings.
│
├── .env
#   - Environment variables file, containing configurations such as project name and Grafana admin credentials.
│
├── build.sh
#   - Script to build and configure the project, typically including commands to set up and start Docker containers.
│
├── docker-compose.yml
#   - Docker Compose file, defining services, networks, and volumes for the Docker containers in the project.
│
├── LICENSE
#   - License file, outlining the terms under which the project's code can be used, modified, and distributed.
│
└── README.md
#   - Documentation file, providing an overview of the project, setup instructions, usage guidelines, and other relevant information.
Follow the explanations provided in the article to populate each file with the required configurations.
3. Proceed to Step 2 below to configure environment variables, customize settings, and start Docker containers.
Option 2: Clone and Customize Pre-made Configuration
If you prefer to start with a pre-configured setup and make final adjustments:
Clone the pre-configured GitHub repository to your local machine:

#!/bin/bash

# Clone the repository from the specified GitHub URL
git clone https://github.com/yourusername/prometheus-grafana-server-monitoring-docker.git

# Move into the cloned repository's directory
cd prometheus-grafana-server-monitoring-docker

# DOCUMENTATION:
# 1. **git clone https://github.com/yourusername/prometheus-grafana-server-monitoring-docker.git**: Clones the repository from the specified GitHub URL into a new directory named `prometheus-grafana-server-monitoring-docker`.
# 2. **cd prometheus-grafana-server-monitoring-docker**: Changes the current directory to `prometheus-grafana-server-monitoring-docker`, allowing you to work within the cloned repository.
# 3. **Usage**: Use these commands to clone the Prometheus and Grafana server monitoring project from GitHub and navigate into the project directory for further configuration or execution.
2. Review and customize the configuration files based on your specific requirements. Make any necessary adjustments to grafana.ini, prometheus.yml, .env, docker-compose.yml, and other files as needed.
3. Skip to Step 3 below to configure any additional settings specific to your environment.
Step 2: Configure Environment Variables
Regardless of which option you chose above, create a .env file in the root directory of your project and add the following:
# Environment Variables from .env file

COMPOSE_PROJECT_NAME=cainam
#   - Defines the project name for Docker Compose, which affects the names of containers and networks.

GF_SECURITY_ADMIN_USER=admin
#   - Sets the Grafana admin username to 'admin' for initial login.

GF_SECURITY_ADMIN_PASSWORD=INSERT_YOUR_DESIRED_STRONG_PASSWORD
#   - Sets the Grafana admin password using an environment variable from the .env file.
#   - Ensure to replace 'INSERT_YOUR_DESIRED_STRONG_PASSWORD' with a secure password of your choice.
Step 3: Customize Grafana Configuration
Edit the grafana/grafana.ini file to suit your needs. For example, you can disable user registration and adjust security settings.
Step 4: Set Up Prometheus Configuration
Edit the prometheus/prometheus.yml file to define your scrape targets and other settings.
Step 5: Build and Start the Docker Containers
Run the build.sh script to build and start the Docker containers:
#!/bin/bash

# Execute the build script
./build.sh

# DOCUMENTATION:
# 1. **./build.sh**: Executes the `build.sh` script located in the current directory.
# 2. **Usage**: Use this command to run the `build.sh` script, which typically contains commands to build and configure the project or application.
This script uses Docker Compose to set up the Grafana and Prometheus services as defined in the docker-compose.yml file.
Step 6: Access Grafana and Add the Node Exporter Dashboard
Once the containers are up and running, follow these steps to access Grafana and add the Node Exporter dashboard using the reference ID:
Open your web browser and go to http://localhost:3000.
Log in using the Grafana admin credentials you set in the .env file.
After logging in, navigate to "Create" > "Dashboard" > "Import".
Enter 1860 as the Grafana.com Dashboard ID. This will import the Node Exporter Full dashboard directly from the Grafana dashboard repository.

What is Node Exporter?
Node Exporter is a Prometheus exporter that collects various system metrics such as CPU usage, memory usage, disk usage, and network statistics from Linux and Unix systems. It allows Prometheus to monitor the performance and health of servers and nodes in your infrastructure.
Node Exporter is essential for gathering data that Grafana can visualize through dashboards, providing insights into system resource utilization and performance trends.
Conclusion
By following these steps, you have successfully set up Grafana and Prometheus using Docker. You can now start monitoring your server infrastructure effectively and visualize metrics using Grafana dashboards, including the Node Exporter Full dashboard.
For more details and to clone the source code, visit the https://github.com/waltervfaustine/prometheus-grafana-server-monitoring-docker.
Feel free to customize and expand this setup according to your needs. Happy monitoring!