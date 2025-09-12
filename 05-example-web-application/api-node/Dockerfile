# Build this docker image with: 
# docker build --target <dev|prod> -t api-node:<version> .

# Pin specific version for stability
# Use slim for reduced image size for better security
FROM node:24-slim AS base

# Specify working directory for the app in the docker container
WORKDIR /usr/src/app

# Copy only files required to install dependencies (better layer caching). This reduces the time to build the image.
COPY package*.json ./

# For target - dev
FROM base AS dev

# --mount=type=cache,target=/usr/src/app/.npm - use a cache mount to speed up the install of existing dependencies. Only additional dependencies not present in the cache will be installed.
# npm set cache /usr/src/app/.npm - set the cache directory to the app directory.
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm &&\
    npm install

# Copy the remaining source code
COPY . .

# Execute the app
CMD ["npm", "run", "dev"]

# For target - production
FROM base AS prod

# Set the environment variable for the app
ENV NODE_ENV=production

# --mount=type=cache,target=/usr/src/app/.npm - use a cache mount to speed up the install of existing dependencies. Only additional dependencies not present in the cache will be installed.
# npm set cache /usr/src/app/.npm - set the cache directory to the app directory.
# npm ci --only=production - clean install only production dependencies, ignore the dev dependencies.
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm &&\
    npm ci --only=production

# Use non-root user for security purposes
USER node

# Copy remaining source code AFTER installing dependencies. 
# Set the non-root user (node) to have the correct file permissions.
COPY --chown=node:node ./healthcheck .
COPY --chown=node:node ./src .

# Indicate expected port
EXPOSE 3000

# Execute the app
CMD ["node","index.js"]