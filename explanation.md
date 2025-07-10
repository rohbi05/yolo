# explanation.md

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
