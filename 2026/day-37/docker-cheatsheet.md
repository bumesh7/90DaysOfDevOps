# 🐳 Docker Cheat Sheet

## 📦 Container Commands
docker run -it nginx /bin/bash     # Run container (interactive)
docker run -d nginx                # Run container (detached)
docker ps                          # List running containers
docker ps -a                       # List all containers
docker stop <id>                   # Stop container
docker rm <id>                     # Remove container
docker exec -it <id> /bin/bash     # Access running container
docker logs <id>                   # View logs

## 🖼️ Image Commands
docker build -t myapp:latest .     # Build image
docker pull nginx                  # Pull image
docker push username/myapp         # Push image
docker tag myapp username/myapp    # Tag image
docker images                      # List images
docker rmi <id>                    # Remove image

## 💾 Volume Commands
docker volume create myvolume      # Create volume
docker volume ls                   # List volumes
docker volume inspect myvolume     # Inspect volume
docker volume rm myvolume          # Remove volume

## 🌐 Network Commands
docker network create mynet        # Create network
docker network ls                  # List networks
docker network inspect mynet       # Inspect network
docker network connect mynet <container>

## ⚙️ Compose Commands
docker compose up                  # Start services
docker compose up -d               # Detached mode
docker compose down                # Stop services
docker compose ps                  # List services
docker compose logs                # View logs
docker compose build               # Build services

## 🧹 Cleanup Commands
docker system prune                # Remove unused data
docker system prune -a             # Remove all unused images
docker volume prune                # Remove unused volumes
docker network prune               # Remove unused networks
docker system df                   # Show disk usage

# 🔥 Remove EVERYTHING unused (containers, images, volumes, networks)
docker system prune -a --volumes

# Remove only stopped containers
docker container prune

# Remove all unused images
docker image prune -a

## 🏗️ Dockerfile Instructions
FROM node:18            # Base image
WORKDIR /app            # Working directory
COPY . .                # Copy files
RUN npm install         # Install dependencies
EXPOSE 3000             # Expose port
CMD ["npm", "start"]    # Default command
ENTRYPOINT ["node"]     # Fixed command

## 🚀 Tips
Use .dockerignore to reduce image size  
Use multi-stage builds for smaller images  
Prefer COPY over ADD  
Use lightweight images (alpine)
