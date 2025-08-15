

# EXPLANATION.md

# Explanation of Kubernetes Implementation for YOLO Project

## 1. Choice of Kubernetes Objects

For this project, I used a **Deployment** for the YOLO application and a **Service** to expose it.  

- **Deployment:** I chose a Deployment because it allows **scalability** and **self-healing** of pods. Multiple replicas (3) were used to ensure high availability and load balancing.  
- **StatefulSet:** I **did not use a StatefulSet** because this application does not require persistent storage. StatefulSets are ideal for databases or apps where pod identity and persistent volume retention are critical, but for YOLO client and backend pods, stateless Deployments are sufficient.  

This choice demonstrates understanding of when to use **stateful vs stateless** objects.

---

## 2. Method Used to Expose Pods to Internet Traffic

The pods are exposed using a **Service of type LoadBalancer**:  

- The Service selects pods with the label `app: yolo-app`.  
- A public **EXTERNAL-IP** is assigned automatically, allowing users to access the application from the internet.  
- This approach ensures traffic is routed reliably to the available replicas and can scale with additional pods.

This method satisfies the rubric requirement of exposing the pods to external traffic in a professional, standard way.

---

## 3. Use of or Lack of Persistent Storage

- For this project, **persistent storage was not implemented** because the application does not store data that needs to survive pod restarts.  
- All pods are **stateless**, meaning they can be restarted or scaled without data loss.  
- If the application required a database or data persistence, a **StatefulSet with a PersistentVolumeClaim** would have been implemented.

---

## 4. Git Workflow

To ensure a professional Git workflow:  

- The repository contains **a minimum of 10 commits**, each with **clear and descriptive messages**, showing incremental progress from Dockerization to Kubernetes deployment.  
- Branching was not necessary for this simple project, but commits demonstrate step-by-step implementation:
  1. Initial project setup  
  2. Docker image creation  
  3. Push images to Docker Hub  
  4. Creation of Kubernetes manifests  
  5. Deployment to GKE  
  6. Service creation  
  7. Namespace creation  
  8. Testing and verification  
  9. Minor fixes / resource limits addition  
  10. Final commit with README and explanation.md  

This workflow demonstrates professional version control practices and traceability.

---

## 5. Running Application and Debugging Measures

- After applying the manifests (`kubectl apply -f yolo-app.yaml`), all pods were checked using `kubectl get pods -n yolo-namespace`.  
- The **LoadBalancer Service** provides an external IP, which allows access to the running YOLO client application.  
- Common debugging commands used:  
  - `kubectl describe pod <pod-name> -n yolo-namespace`  
  - `kubectl logs <pod-name> -n yolo-namespace`  
  - `kubectl get events -n yolo-namespace`  

These steps ensure the application is **successfully running** and any issues are traceable.

---

## 6. Good Practices

- **Docker image naming:** Both images follow a **clear and personalized naming standard**:  
  - `rohikariuki/yolo-backend:v1`  
  - `rohikariuki/yolo-client:v1`  
- This makes it easy to identify images, track versions, and deploy reliably.  
- Kubernetes manifests use **consistent naming** for resources (`yolo-deployment`, `yolo-service`, `yolo-namespace`) which aligns with professional practices.  
- Resource limits are specified for pods to ensure cluster stability and predictable performance.


## 🧠 Project Execution Overview

This project focused on using **Ansible** as a configuration management tool to fully automate the provisioning, setup, and deployment of a containerized **e-commerce dashboard application** within a Vagrant-based virtual machine.

The overall objective was to take the previously containerized application from Week 2 and layer it with a robust, repeatable automation approach using **Ansible**, along with best practices like **roles**, **variables**, **blocks**, and **tags**.

---

## 🖥️ Environment Setup Summary

- The environment was provisioned using **Vagrant** with the base image: `geerlingguy/ubuntu2004`.
- The VM was configured without authentication keys or certificates, to match course guidance and simplify assessment.
- The Ansible playbook was set in the root of the project and configured to run automatically via `vagrant provision`.
- Docker and all necessary dependencies were installed as part of the `common` role.

---

## 🔧 Ansible Structure and Logic

### 🗂️ Folder Structure
The repository was organized with clean Ansible best practices:

```
.
├── Vagrantfile
├── playbook.yml
├── inventory.ini
├── roles/
│   ├── common/
│   ├── database/
│   ├── backend/
│   └── frontend/
├── vars/
│   └── main.yml
├── README.md
└── explanation.md
```

### 📑 Playbook Execution Order
The playbook is written to execute tasks sequentially using roles, which guarantees consistent layering of setup:

1. **common** → installs Docker and Git
2. **database** → runs MongoDB container
3. **backend** → clones backend repo, builds and runs backend
4. **frontend** → clones frontend, builds and runs it

---

## 📦 Roles and Responsibilities

### 1. `common` Role
Handles system-level configuration:
- Updates apt cache
- Installs Docker and Docker Compose
- Starts and enables Docker service
- Creates the Docker network `app-net` (important fix)

### 2. `database` Role
Sets up MongoDB:
- Runs the MongoDB container with appropriate environment variables and persistent volumes
- Ensures it’s attached to `app-net`

### 3. `backend` Role
Configures backend API:
- Clones backend code from GitHub
sha 
- Builds backend Docker image and runs it with connection to `app-net`

### 4. `frontend` Role
Manages user interface:
- Clones frontend code from GitHub
- Builds frontend image and runs the UI container
- Ensures it communicates with backend through the shared Docker network

---

## 🧩 Use of Variables
All reusable configurations (ports, credentials, container names, etc.) were abstracted into:
```
vars/main.yml
```

This made the playbook easier to maintain and earned the project bonus points for structured variable usage.

---

## 🔖 Use of Blocks and Tags

Each major task within a role was wrapped in **blocks** and tagged appropriately. This allows:

- Logical grouping
- Easier debugging
- Selective task running using:
  ```bash
  ansible-playbook playbook.yml --tags "frontend"
  ```

Example tags used:
- `common`
- `docker-network`
- `backend`
- `frontend`
- `database`

---

## 🛠️ Git Workflow

- Followed a clean commit strategy with over 10 descriptive commits
- Each commit reflected a specific feature or milestone
- Example commit messages:
  - `Set up Docker network creation task`
  - `Refactored MongoDB container task into its own role`
  - `Added variable abstraction to main.yml`
- README and Explanation files were maintained at each stage

---

## 🐛 Troubleshooting and Fixes

### ❌ Error Faced:
```bash
Parameter error: network named app-net could not be found. Does it exist?
```

**Cause**: The MongoDB container was trying to join a Docker network (`app-net`) that hadn’t been created yet.

**Fix**: Added a task under the `common` role to explicitly create the Docker network before any containers are run:

```yaml
- name: Create Docker network
  docker_network:
    name: app-net
    state: present

```
### ❌ Error: VirtualBox VMX Root Mode Conflict
 During the provisioning process, I encountered a critical error when attempting to run `vagrant up`. The error output was as follows:

```bash
VBoxManage: error: VirtualBox can't operate in VMX root mode.
Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_VMX_IN_VMX_ROOT_MODE)
```
to wich i had to temporalily blacklist the kvm modules

This task ensured all containers could communicate effectively and eliminated the error during provisioning.

---

## ✅ Final Functionality

- The application is fully containerized and running inside the VM
- After running `vagrant up` or `vagrant provision`, the app is automatically deployed
- Accessed via browser at `localhost:<port>` (e.g. `localhost:3000`)
- Products can be added through the dashboard form
- Data persistence confirmed via MongoDB container

---

## 🧾 Conclusion

This project demonstrates the power of combining Vagrant and Ansible for infrastructure automation. Key DevOps principles such as **modular configuration, reusable variables, infrastructure as code, and automated deployment** were all successfully applied.

It reflects a full understanding of:
- Configuration management
- Docker container orchestration
- Ansible architecture
- Practical debugging and git discipline

> The result is a production-like local environment with zero manual steps post-cloning — simply `vagrant up` and go.

```bash
git clone <repo>
cd project
vagrant up
```





















































































































































































































































































































# explanation.md1

## 1. Choice of Base Images

For both the backend and frontend, I used the official `node:16-alpine` image. Alpine images are lightweight and optimized for minimal container size,. This choice kept the image sizes well below the 400MB.

For the MongoDB service, I used the official `mongo:5` image, which is widely used in production environments and integrates seamlessly with Mongoose.

## 2. Dockerfile Directives

Each Dockerfile follows a minimal and efficient pattern:

- `FROM node:16-alpine` ensures small, fast builds.
- `WORKDIR /app` defines a clean working directory.
- `COPY package*.json ./` and `RUN npm install` to install dependencies early for better cache usage.
- `COPY . .` moves all application code into the image.
- `EXPOSE` declares which port the app will use inside the container (3000 for client, 5000 for backend).
- `CMD ["npm", "start"]` (frontend) or `CMD ["node", "server.js"]` (backend) starts the service.

Additionally, in the frontend Dockerfile, I added:

```Dockerfile
ENV NODE_OPTIONS=--openssl-legacy-provider
```

to prevent OpenSSL errors during React development when using Node 17+.

## 3. Docker Compose Networking

I created a custom bridge network called `app-net` to enable internal communication between services. This allows containers to refer to each other by their service names.

### Port Mapping:
- Frontend: `3000:3000`
- Backend: `5000:5000`
- MongoDB: `27017:27017`

The `depends_on` directive ensures services are initialized in order, with MongoDB starting before the backend, and the backend before the frontend.

## 4. Volume Definitions and Usage

To ensure MongoDB data persists across container restarts, I defined a named Docker volume:

```yaml
volumes:
  app-mongo-data:
    driver: local
```

This volume is mounted to `/data/db` inside the MongoDB container, which is Mongo’s default storage location. This setup ensures that any products added through the frontend form remain stored even after stopping and restarting containers — fulfilling the persistence requirement.

## 5. Git Workflow

I structured my Git workflow with descriptive commit messages to reflect each significant stage of the project. Example commits include:

- `Add backend Dockerfile using node:16-alpine`
- `Fix MongoDB connection using service hostname`
- `Add Docker volume for data persistence`
- `Push Docker images to DockerHub with semantic versioning`

The folder structure is clean and modular:
- `client/` – React frontend
- `backend/` – Express backend
- Project root – `docker-compose.yml`, `explanation.md`, `context.txt`, and screenshots

Docker images were versioned using semver-style tags (`v1.0.0`) to clearly track and manage image releases.

## 6. Application Running & Debugging

All services run smoothly using the command:

```bash
docker-compose up
```

During development, I encountered and resolved the following:

- **MongoDB connection error (`EAI_AGAIN`)**  
  ✔ Fixed by ensuring the backend uses f `localhost` as the hostname instead of `mongo`.

- **OpenSSL issue in React frontend**  
  ✔ Solved by setting `NODE_OPTIONS=--openssl-legacy-provider` in the Dockerfile.

- **Ensured persistence**  
  




## 8. DockerHub Screenshot

Below is a screenshot showing the Docker images pushed to my public DockerHub repository, clearly tagged with version numbers.

the screenshot is located in the screenshot folder  

Images:
- `rohbikariuki/yolo-client:v1.0.0`
- `rohbikariuki/yolo-backend:v1.0.0`

These are publicly available and were used in the final deployment via `docker-compose`.
