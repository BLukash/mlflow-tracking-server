# MLflow Tracking Server template (Docker + SQLite + S3)

This setup is a template to run a lightweight **MLflow Tracking Server** inside Docker on an EC2 instance.  

- **Metadata (runs, params, metrics):** stored in a local SQLite DB on an EBS-backed Docker volume (can be synced to s3 via crontab, example below).
- **Artifacts (models, logs, plots):** stored in Amazon S3.

---

## Quick Start

1. **Launch EC2 Instance**
    - Use Ubuntu or Amazon Linux.
    - Security Group: open TCP port 5000 (only to your IP (not 0.0.0.0/0) if ypu value security).
    - IAM Role: attach to the EC2 instance with S3 permissions (or just S3 full access):
        - s3:PutObject
        - s3:GetObject
        - s3:ListBucket

2. **Create S3 bucket for artifacts storage and metadata backup**

3. **[Install Docker and Compose](https://docs.docker.com/engine/install/ubuntu/)**
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    sudo usermod -aG docker $USER
    ```

4. **Clone repo**
    ```bash
    git clone https://github.com/BLukash/mlflow-tracking-server.git
    cd mlflow-tracking-server
    ```

4. **Set Environment Variables**
    Create a .env file in the project root:
    ```env
    AWS_DEFAULT_REGION=<your-region>
    MLFLOW_ARTIFACT_URI=s3://<your-mlflow-artifacts-bucket>/mlruns
    ```

5. **Build Docker images and start mlflow tracking server**
    ```bash
    docker compose build
    docker compose up -d
    ```

6. **Access UI**
    - http://<EC2-public-IP>:5000

7. **Automatic Backup of SQLite DB via crontab (every Sunday at 4:20, overriding previous backup)**
    ```bash
    crontab -e
    20 4 * * 0 docker exec mlflow aws s3 cp /mlflow/db/mlflow.db s3://my-mlflow-artifacts-bucket/backups/mlflow-weekly.db
    ```