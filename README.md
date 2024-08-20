# Enhancing Server Performance and Reliability: A Guide to Installing and Using Prometheus and Grafana for Server Monitoring using Docker


Monitoring your server infrastructure is crucial for maintaining the health and performance of your applications. Grafana and Prometheus are two powerful tools that can help you achieve this. In this article, we will walk you through the process of setting up Grafana and Prometheus using Docker. By the end of this guide, you will have a fully functional monitoring setup.
Project Folder Structure
Before we dive into the setup process, let's look at the folder structure you will need. Each file and folder has a specific role in the setup:

```
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
```

## Folder Structure

### 1. `grafana/`

This directory contains configuration files for Grafana.

- **`grafana.ini`**: This is the main configuration file for Grafana. It allows you to customize various settings such as security, data sources, and logging.

#### Sample Grafana Configuration File

**File: `grafana/grafana.ini`**

```ini
# [server] section: Configuration related to the Grafana server
[server]
http_port = 3000  # The port Grafana will listen on. Default is 3000.

# [security] section: Configuration related to Grafana security settings
[security]
admin_user = admin  # The username for the Grafana admin user. Default is 'admin'.
admin_password = ${GF_SECURITY_ADMIN_PASSWORD}  # The password for the Grafana admin user. Uses an environment variable for security.
```


### 2. `prometheus/`

This directory holds the configuration files for Prometheus.

- **`prometheus.yml`**: This file contains the configuration for Prometheus, including global settings and scrape configurations that define the metrics sources.

#### Sample Prometheus Configuration File

**File: `prometheus/prometheus.yml`**

```yaml
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
      - targets: ['icodebible-node-exporter:9100']  # Target for Node Exporter metrics, typically the Node Exporter container
```

### 3. `.env`

The `.env` file is used to define environment variables that are utilized in your Docker setup. These variables can be referenced in the Docker Compose file to manage configuration settings such as usernames and passwords securely.

#### Sample `.env` File

**File: `.env`**

```dotenv
# Environment Variables for Docker Setup

COMPOSE_PROJECT_NAME=icodebible
#   - Defines the project name for Docker Compose, which affects the names of containers and networks.

GF_SECURITY_ADMIN_USER=admin
#   - Sets the Grafana admin username to 'admin' for initial login.

GF_SECURITY_ADMIN_PASSWORD=INSERT_YOUR_DESIRED_STRONG_PASSWORD
#   - Sets the Grafana admin password using an environment variable from the .env file.
#   - Ensure to replace 'INSERT_YOUR_DESIRED_STRONG_PASSWORD' with a secure password.
```

### 4. `build.sh`

This is a shell script used to build and manage your Docker Compose setup. It contains commands to bring up and tear down the Docker services.

#### Sample `build.sh` Script

**File: `build.sh`**

```bash
#!/bin/bash
# Docker Compose Command for Deployment Management

# Move to the project root directory
cd ..

# Stop and remove all running containers defined in the Docker Compose file
docker-compose -f docker-compose.yml down

# Build images, recreate containers if necessary, and start all containers in detached mode
docker-compose -f docker-compose.yml up --build -d
```


### 5. `docker-compose.yml`

The `docker-compose.yml` file defines the Docker services for Grafana and Prometheus. It specifies the images to use, the ports to expose, and the volumes to mount for configuration files.

#### Sample `docker-compose.yml` Configuration

**File: `docker-compose.yml`**

```yaml
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
    container_name: icodebible-prometheus
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
      - icodebible-network
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
    container_name: icodebible-grafana
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
      - icodebible-network

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
    container_name: icodebible-node-exporter
    # Map host machine's port 9100 to the container's port 9100
    ports:
      - "9100:9100"

    depends_on:
      - grafana # Depend on Grafana service for proper startup order
      - prometheus # Depend on Prometheus service for proper startup order

    # Attach the service to the monitoring network
    networks:
      - icodebible-network

# Define networks for the services
networks:
  icodebible-network:
    external: true # Use an external network named 'icodebible-network'
```

## Step-by-Step Configuration

### Step 1: Choose Your Configuration Approach

You have two approaches to set up Grafana and Prometheus:

#### Option 1: Prepare Configuration from Scratch

If you prefer to set up Grafana and Prometheus from scratch using the information provided in this guide:

1. **Create a New Directory**

   First, create a new directory for your project and navigate into it:

   ```bash
   # Create a new directory named `prometheus-grafana-server-monitoring-docker`
   mkdir prometheus-grafana-server-monitoring-docker

   # Move into the newly created directory
   cd prometheus-grafana-server-monitoring-docker
    ```


2. Set up the folder structure and configuration files as outlined below:

    ```yaml
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
    ```

Follow the explanations provided in the article to populate each file with the required configurations.

3. Proceed to Step 2 below to configure environment variables, customize settings, and start Docker containers.

#### Option 2: Clone and Customize Pre-made Configuration
If you prefer to start with a pre-configured setup and make final adjustments:
1. Clone the pre-configured GitHub repository to your local machine:

    ```bash
    #!/bin/bash

    # Clone the repository from the specified GitHub URL
    git clone https://github.com/yourusername/prometheus-grafana-server-monitoring-docker.git

    # Move into the cloned repository's directory
    cd prometheus-grafana-server-monitoring-docker
    ```


2. Review and customize the configuration files based on your specific requirements. Make any necessary adjustments to grafana.ini, prometheus.yml, .env, docker-compose.yml, and other files as needed.
3. Skip to Step 3 below to configure any additional settings specific to your environment.

#### Step 2: Configure Environment Variables
Regardless of which option you chose above, create a .env file in the root directory of your project and add the following:

  ```dotenv
  # Environment Variables from .env file

  COMPOSE_PROJECT_NAME=icodebible
  #   - Defines the project name for Docker Compose, which affects the names of containers and networks.

  GF_SECURITY_ADMIN_USER=admin
  #   - Sets the Grafana admin username to 'admin' for initial login.

  GF_SECURITY_ADMIN_PASSWORD=INSERT_YOUR_DESIRED_STRONG_PASSWORD
  #   - Sets the Grafana admin password using an environment variable from the .env file.
  #   - Ensure to replace 'INSERT_YOUR_DESIRED_STRONG_PASSWORD' with a secure password of your choice.
  ```

#### Step 3: Customize Grafana Configuration
Edit the grafana/grafana.ini file to suit your needs. For example, you can disable user registration and adjust security settings.

#### Step 4: Set Up Prometheus Configuration
Edit the prometheus/prometheus.yml file to define your scrape targets and other settings.

#### Step 5: Build and Start the Docker Containers
Run the build.sh script to build and start the Docker containers:

  ```bash
  #!/bin/bash

  # Execute the build script
  ./build.sh
  ```
This script uses Docker Compose to set up the Grafana and Prometheus services as defined in the docker-compose.yml file.

#### Step 6: Access Grafana and Add the Node Exporter Dashboard
Once the containers are up and running, follow these steps to access Grafana and add the Node Exporter dashboard using the reference ID:
1. Open your web browser and go to http://localhost:3000.
2. Log in using the Grafana admin credentials you set in the .env file.
3. After logging in, navigate to "Create" > "Dashboard" > "Import".
4. Enter 1860 as the Grafana.com Dashboard ID. This will import the Node Exporter Full dashboard directly from the Grafana dashboard repository.

### What is Node Exporter?
Node Exporter is a Prometheus exporter that collects various system metrics such as CPU usage, memory usage, disk usage, and network statistics from Linux and Unix systems. It allows Prometheus to monitor the performance and health of servers and nodes in your infrastructure.
Node Exporter is essential for gathering data that Grafana can visualize through dashboards, providing insights into system resource utilization and performance trends.

### Conclusion
By following these steps, you have successfully set up Grafana and Prometheus using Docker. You can now start monitoring your server infrastructure effectively and visualize metrics using Grafana dashboards, including the Node Exporter Full dashboard.

For more details and to clone the source code, visit the https://github.com/waltervfaustine/prometheus-grafana-server-monitoring-docker.

Feel free to customize and expand this setup according to your needs. Happy monitoring!