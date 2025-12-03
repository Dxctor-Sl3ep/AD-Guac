🚀 Apache Guacamole on Docker: Why this Project?

This project deploys Apache Guacamole — a clientless remote desktop gateway — using Docker Compose.

The main goal is to provide a secure, easy-to-manage remote access solution while leveraging the benefits of containerization.

🥑 1. Why Apache Guacamole

Guacamole is like a VPN for remote server access. It centralizes your connections (RDP, SSH, VNC) and lets you access them from a web browser without installing special client software.

    Web access: connect to your servers from any device (PC, tablet, phone) through a browser.

    Centralization: a single access point and a single open port (80 or 443) can serve all protocols.

    Security: Guacamole handles authentication, strengthening access protection.

    Auditing: it logs connections and can record sessions for full traceability.


🐳 2. Why Containerization (Docker Compose)

Deploying Guacamole with Docker Compose solves dependency, environment, and complex service management issues.

2.1 Isolation and Environment

    Zero dependency conflicts: Guacamole requires Java and various build tools (guacd). Docker packages all required dependencies inside containers.

    Consistent environment: the service will behave the same whether you run it on Windows, macOS, or Linux.

2.2 Ease of Deployment and Scalability

    One-command deployment: the `docker-compose.yml` defines the whole architecture (guacamole, guacd, and mariadb). Start with `docker compose up -d`.

    Modular architecture: each service runs in its own container:

        - `guacamole_db`: stores users and connections.
        - `guacd`: the daemon that handles protocols (RDP/VNC/SSH).
        - `guacamole`: the web frontend.

    Data persistence: Docker volumes ensure database data and session recordings persist even if containers are recreated or updated.

2.3 Management and Maintenance

    Centralized configuration: critical variables (passwords, database names) are managed via a `.env` file.

    Simplified upgrades: to update Guacamole, change the image tag in `docker-compose.yml` and restart the stack (`docker compose pull` then `docker compose up -d`).


📖 3. Getting Started

All steps to run this stack are detailed in the installation guide: [installation](./install.md)
