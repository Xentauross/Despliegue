# syntax=docker/dockerfile:1.4

#  Base development environment
FROM --platform=$BUILDPLATFORM node:lts AS development

WORKDIR /code
# Copy dependency files first to leverage Docker's layer caching
COPY package.json /code/package.json
COPY package-lock.json /code/package-lock.json

# Clean install of dependencies based on the lockfile
RUN npm ci
# Copy the rest of the application source code
COPY . /code

# Environment variables for Continuous Integration and default port
ENV CI=true
ENV PORT=3000

CMD [ "npm", "start" ]

#Enhanced development environment 
FROM development AS dev-envs
RUN <<EOF
apt-get update
apt-get install -y git
EOF

# Set up a non-root user 'vscode' and give it access to the Docker group
RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF

# Copy Docker CLI tools from an external image to enable Docker-out-of-Docker usage
COPY --from=gloursdocker/docker / /
CMD [ "npm", "start" ]

#Build the production assets
FROM development AS build
# Generates the static files 
RUN ["npm", "run", "build"]

#Production server
FROM nginx:1.13-alpine

# Copy only the compiled static files from the build stage to the Nginx web root
COPY --from=build /code/build /usr/share/nginx/html