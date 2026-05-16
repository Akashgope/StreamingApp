# StreamingApp

Stream premium video content, host live watch parties, and manage your catalogue with a modern microservice architecture. The platform now ships with a production-ready admin portal, real-time chat, S3-backed adaptive streaming, and a redesigned cinematic frontend experience.

## Architecture

| Service | Port | Description |
| --- | --- | --- |
| `authService` | 3001 | User authentication, registration, JWT issuance |
| `streamingService` | 3002 | Video catalogue, S3 playback endpoints, public APIs |
| `adminService` | 3003 | Dedicated admin microservice for asset management and uploads |
| `chatService` | 3004 | Websocket + REST chat for live watch parties |
| `frontend` | 3000 | React SPA with revamped UI and integrated chat |
| `mongo` | 27017 | Shared MongoDB instance |

All backend services share common database models and utilities through `backend/common`.

## Environment Configuration

Create an `.env` for each service (or export variables before running). All services accept the standard AWS credentials for S3 access.

### Auth Service (`backend/authService/.env`)
```ini
PORT=3001
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Streaming Service (`backend/streamingService/.env`)
```ini
PORT=3002
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
AWS_CDN_URL=
STREAMING_PUBLIC_URL=http://localhost:3002
```

### Admin Service (`backend/adminService/.env`)
```ini
PORT=3003
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Chat Service (`backend/chatService/.env`)
```ini
PORT=3004
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
```

### Frontend build variables (`frontend/.env` or Docker build args)
```ini
REACT_APP_AUTH_API_URL=http://localhost:3001/api
REACT_APP_STREAMING_API_URL=http://localhost:3002/api
REACT_APP_STREAMING_PUBLIC_URL=http://localhost:3002
REACT_APP_ADMIN_API_URL=http://localhost:3003/api/admin
REACT_APP_CHAT_API_URL=http://localhost:3004/api/chat
REACT_APP_CHAT_SOCKET_URL=http://localhost:3004
```

## Running with Docker Compose

1. Populate the environment variables above (or rely on the defaults baked into `docker-compose.yml`).
2. Build and start the stack:
   ```bash
   docker-compose up --build
   ```
3. Navigate to `http://localhost:3000` for the web app.

The compose file provisions MongoDB plus all four Node.js microservices. S3 credentials are optional for local testing—you can still browse seeded metadata, but streaming requires valid S3 objects.

## Local Development

Install dependencies for each service:

```bash
# auth service
cd backend/authService && npm install

# streaming service
cd ../streamingService && npm install

# admin service
cd ../adminService && npm install

# chat service
cd ../chatService && npm install

# frontend
cd ../../frontend && npm install
```

Run the services (in separate terminals) after starting MongoDB:

```bash
cd backend/authService && npm run dev
cd backend/streamingService && npm run dev
cd backend/adminService && npm run dev
cd backend/chatService && npm run dev
cd frontend && npm start
```

## Feature Highlights

- **S3-backed adaptive streaming** with secure signed uploads for admins.
- **Dedicated admin microservice** for video ingestion, metadata management, and featured curation.
- **Real-time chat** overlay in the player (Socket.IO + persistent message history).
- **Modern React experience** featuring cinematic hero sections, dynamic carousels, and responsive design.
- **Role-aware access control** across frontend routes and backend microservices.

## Testing

Automated tests are not yet included. Recommended smoke checks:

1. Register and log in through the web UI.
2. Upload a small video + thumbnail via the admin dashboard (requires valid S3 credentials).
3. Confirm playback from the browse page and verify that chat messages broadcast between multiple browser tabs.

## License

MIT © StreamFlix Team
# Trigger build



**What I Did to Complete This Project**

1. Forked and Cloned the Repository
Forked the original repository UnpredictablePrashant/StreamingApp into my own GitHub account (Akashgope/StreamingApp).

Cloned the fork locally:

bash
git clone https://github.com/Akashgope/StreamingApp.git
cd StreamingApp
Added the original as an upstream remote to keep my fork in sync.

2. Created Dockerfiles for Containerization
Frontend (frontend/Dockerfile) – multi‑stage build: Node 18 to compile React, then Nginx to serve static files. Added ENV NODE_OPTIONS="--max-old-space-size=512" to avoid memory issues.

Backend (backend/Dockerfile) – generic Dockerfile that takes a SERVICE_NAME build argument. It copies the respective microservice folder (authService, streamingService, etc.) and the common folder, runs npm ci, and starts the service.

Created an empty backend/common folder (with a .gitkeep file) to satisfy the Dockerfile’s copy instruction.

Pushed all changes to GitHub.

3. Set Up AWS ECR Repositories
Configured AWS CLI with my access keys.

Created five ECR repositories using the AWS CLI:

bash
aws ecr create-repository --repository-name streamingapp-frontend --region us-east-1
aws ecr create-repository --repository-name streamingapp-backend-auth --region us-east-1
# ... (streaming, admin, chat)
Authenticated Docker to ECR and manually built/pushed images to verify everything worked.

4. Built a Jenkins CI Pipeline on an EC2 Instance
Launched a t3.micro EC2 instance (later upgraded to t3.small due to memory constraints) with Ubuntu 22.04.

Installed Docker and ran Jenkins inside a container, mounting the host’s Docker socket.

Inside the Jenkins container, installed awscli and copied the docker binary from the host.

Added my AWS credentials (Access Key ID + Secret Access Key) as Jenkins credentials.

Created a Pipeline job named StreamingApp-CI that uses the Jenkinsfile from my GitHub repository.

Configured a GitHub webhook to trigger Jenkins on every push to the main branch.

The Jenkinsfile performs:

Checkout code

Login to ECR

Build and push frontend image

Build and push all backend images (auth, streaming, admin, chat) with lowercase repo names.

Successfully ran the pipeline – all images are now stored in ECR automatically on every push.

<img width="3046" height="1166" alt="image" src="https://github.com/user-attachments/assets/be8ec340-b5fa-4453-b76f-6158fbf5911f" />

<img width="1539" height="517" alt="Screenshot 2026-05-16 at 8 41 46 PM" src="https://github.com/user-attachments/assets/0a0d5b9e-f0ee-4af8-9046-22ef814f8ef8" />

<img width="1549" height="746" alt="Screenshot 2026-05-16 at 8 53 10 PM" src="https://github.com/user-attachments/assets/5ea24e3a-a3ca-42c9-a693-e0a1d80df5fd" />


5. Created an Amazon EKS Cluster
Installed kubectl, eksctl, and helm on my local machine.

Used eksctl to create a cluster with managed nodegroup:

bash
eksctl create cluster --name streaming-cluster --region us-east-1 --nodegroup-name streaming-nodes --node-type t3.medium --nodes 2 --managed
Verified nodes were ready with kubectl get nodes.

<img width="570" height="132" alt="Screenshot 2026-05-16 at 9 16 16 PM" src="https://github.com/user-attachments/assets/c9518b3a-e0c7-486b-a8d3-16b3f096d5d8" />


6. Wrote Helm Charts for the MERN Stack
Created a Helm chart directory: helm/streamingapp/ with Chart.yaml, values.yaml, and templates/.

In templates/, created YAML files for:

frontend – Deployment and LoadBalancer Service.

auth, streaming, admin, chat – each with Deployment and ClusterIP Service.

mongodb – Deployment and Service to provide a database for the backend services.

Set environment variables (MONGODB_URI, JWT_SECRET, PORT) for each backend so they can connect to MongoDB.

Deployed the chart:

bash
helm install streamingapp ./helm/streamingapp --namespace streaming-ns --create-namespace
Checked that all pods became Running:

bash
kubectl get pods -n streaming-ns
<img width="570" height="194" alt="Screenshot 2026-05-16 at 9 15 38 PM" src="https://github.com/user-attachments/assets/5fffbce1-acff-4951-b8bf-8d12b7a7123c" />


7. Accessed the Application
Retrieved the frontend LoadBalancer external hostname:

bash
kubectl get svc -n streaming-ns frontend -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
Opened the URL in a browser – the streaming app loaded successfully.

<img width="1554" height="900" alt="Screenshot 2026-05-16 at 9 30 48 PM" src="https://github.com/user-attachments/assets/fae3245a-d5e3-4a46-b1fb-0d59524a5fd9" />


8. Enabled Monitoring with CloudWatch Container Insights
Enabled cluster logging and installed the CloudWatch observability add‑on:

bash
eksctl utils update-cluster-logging --enable-types all --region us-east-1 --cluster streaming-cluster --approve
eksctl create addon --name amazon-cloudwatch-observability --cluster streaming-cluster --region us-east-1
Went to the CloudWatch Console → Container Insights to view cluster metrics.

<img width="1132" height="576" alt="image" src="https://github.com/user-attachments/assets/bfb0bfd0-71b7-4b4d-a592-a600cfdb4ff1" />

9. Cleaned Up Resources (after submission)
Deleted the Helm release, the EKS cluster, and terminated the Jenkins EC2 instance to avoid ongoing charges.

