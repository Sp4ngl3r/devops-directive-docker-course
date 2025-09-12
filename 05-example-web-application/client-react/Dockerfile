FROM node:19.4-bullseye AS build

# Specify working directory other than /
WORKDIR /usr/src/app

# Copy only files required to install
# dependencies (better layer caching)
COPY package*.json ./

# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci

# Copy the rest of the application code
COPY . .

# Build the application
RUN npm run build 

# Use separate stage for deployable image
FROM nginxinc/nginx-unprivileged:alpine3.22-perl

# Use COPY --link to avoid breaking cache if we change the second stage base image
COPY --link nginx.conf /etc/nginx/conf.d/default.conf

# Copy the build output to replace the default nginx contents
# After running the build, the dist directory will be created in the build stage, which contains the static files for the application
# Store these files in the default nginx html directory
COPY --link --from=build /usr/src/app/dist/ /usr/share/nginx/html

EXPOSE 8080