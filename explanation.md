YOLO E-Commerce — Explanation Document
Overview
This document explains the YOLO E-Commerce full stack web app, built using MERN-stack and composed of three microservices: frontend, backend (Node/Express) and database (MongoDB).

The project is containerized with Docker and orchestrated using Docker Compose.

Base Image Choice
Backend :

~ used a multi-stage build using node:10-alpine building stanfe and used alpine:3.16.7for runtime stage for a lightweight node environment keeping the final backend image size to 86.9MB. The Alpine variants ensures a smaller footprint and faster building times.

~ Include npm for dependency installation.

Frontend :

~ I used a multi-stage build as well, using node:14-slim for the building stage and nginx:alpine for the runtime stage. The build image compiles the React code, and only the final static files are copied to the container. This reduces the final image size to just 54.8MB, ensuring efficient serving of the frontend.

Database :

~ Used mongo:3.0, which is quite small and provides a stable and production-ready MongoDB setup. This builds a total image size of the database container to 232MB, which is was required.

Dockerfiles
Both backend and frontend Dockerfiles uses multi-stage builds to separate build and runtime stages for smaller and cleaner final images.

Backend
Used RUN npm install --prod --omit=dev to install only production dependencies and ommit test and test and dev dependancies if any.

Defined WORKDIR /usr/src/app for project structure.

Copied source code and launched the app with CMD ["node", "server.js"].

Frontend(Client)
Built the app using npm run build in the build stage.

Copied only the build dir to Nginx (/usr/share/nginx/html).

Used CMD ["nginx", "-g", "daemon off;"] to keep Nginx running in the foreground

database
Pulls mongoDB image that will be use to create a custome database image.
Docker Compose Networking
All microservices (frontend, backend, and database) communicate over a custom bridge network defined in docker-compose.yml.

backend-db-net: connects backend and database. This allows communication between services by service name,e.g., the backend connects to MongoDB via mongodb://mongo:27017/yolo-db.

frontend-backend-net: connects clinet to backend. Frontend communicate with backend which communicates with database.

There is no direct communication of frontend and database

Port mappings/binding:
Frontend:8080:80

Backend: 5000:5000

Database: 27017:27017

Docker Volume
Database volume
I named database volume mongo_data and mounted at /data/db in the database container to persist product data even after container shutdown.

Frontend volume
Named as client-logs and mounted at /usr/src/app/logs to gather logs.

Backend volume
Named as backend-logs and mounted at /usr/src/app/logs to gather logs.

Running and Debugging
Running
Running docker-compose up --build will successfully launch all containers.

Visiting http://localhost:8080 loads the e-commerce web app, and adding products using the “Add Product” form will confirms backend–database connectivity.

After restarting the containers, and still see products you added, it verifies persistent storage through the defined volume.

Debugging
I used docker logs container_name/container_id to monitor output and docker exec -it container_name/container_id sh to inspect running containers interactively in a shell.


I pushed Images to DockerHub using:(login to docker hub using docker login and follow instruciton to log in.)

        docker push kibet6/kibet-backend
        docker push kibet6/kibet-client
        docker push kibet6/kibet-database


Quick start (local, Docker Compose)
Build and start all services (from the root directory):
docker-compose up --build
Run in detached mode:
docker-compose up -d --build
Stop and remove containers, networks and volumes:
docker-compose down -v
View logs:
docker-compose logs -f
to view last 50 logs
docker-compose logs -f --tail 50
verify containers are running
docker ps -a 
confirm custom networks created
docker network ls  
confirm volumes are persistent
docker volume ls    
confirm images are created
docker image ls    


Summary
The final containerized application runs successfully via Docker Compose command.

All components are properly orchestrated through custom bridge networks, with persistent data handled via Docker volumes.

The use of lightweight base images, semantic and tagging ensures clean, efficient, and reproducible builds.

Final Image Sizes:

Backend: 33.2MB

Frontend: 54.8MB

Database: 232MB

Total combined size is under the 400MB

Author & License
Author: Victor Kibet

The original project was forked from yolo.