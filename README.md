# Jenkins Master-Agent Setup with Docker Compose

This README provides a step-by-step guide for setting up a Jenkins agent (slave) using Docker Compose to connect with a Jenkins master server running on a virtual machine (VM).

## Prerequisites
- Jenkins master server running on a VM.
- Docker and Docker Compose installed on the host machine.
- Open ports on the VM for communication:
  - **8080**: For Jenkins web interface.
  - **50000**: For JNLP agent communication.

---

## Configuration Overview
### 1. Jenkins Master Setup
1. Log in to the Jenkins master.
2. Navigate to **Manage Jenkins** > **Configure Global Security**:
   - Set **TCP port for inbound agents** to `Fixed` (e.g., `50000`).
   - Ensure **JNLP4** is enabled in **Agent Protocols**.
3. Save the configuration.

4. Create a new node:
   - Go to **Manage Jenkins** > **Manage Nodes and Clouds** > **New Node**.
   - Configure the node:
     - **Node Name**: `agent`
     - **Launch Method**: `Launch agents via Java Web Start (JNLP)`.
   - Save the node and copy the agent secret.

---

### 2. Docker Compose File
Create a `docker-compose.yml` file with the following content:

```yaml

services:
  jenkins-agent:
    image: jenkins/inbound-agent
    container_name: jenkins-agent
    restart: always
    environment:
      - JENKINS_URL=http://<VM-IP-ADDRESS>:8080/
      - JENKINS_SECRET=<YOUR_AGENT_SECRET>
      - JENKINS_AGENT_NAME=agent
      - JENKINS_AGENT_WORKDIR=/home/jenkins/agent
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./jenkins_agent:/home/jenkins/agent
    networks:
      - jenkins-network

networks:
  jenkins-network:
    driver: bridge
```

Replace:
- `<VM-IP-ADDRESS>` with the IP address of your Jenkins master VM.
- `<YOUR_AGENT_SECRET>` with the agent secret obtained from Jenkins.

---

### 3. Start the Agent
Run the following commands to start the agent:

```bash
docker-compose up -d
```

Monitor the logs:

```bash
docker logs -f jenkins-agent
```

The agent should connect to the master and appear as **Online** in the Jenkins **Manage Nodes** section.
