# Build this docker image with: 
# docker build --target <dev|prod> -t api-golang:<version> .

# Pin specific version for stability
# Use alpine for smaller image size
FROM golang:1.24.6-bullseye AS build-common

# Update package lists and apply security updates for Kerberos vulnerabilities
RUN apt-get update && apt-get upgrade -y \
    libgssapi-krb5-2 \
    libk5crypto3 \
    libkrb5-3 \
    libkrb5support0 \
    && rm -rf /var/lib/apt/lists/*

# Specify working directory for the app in the docker container
WORKDIR /app

# Copy only files required to install dependencies (better layer caching). This reduces the time to build the image.
COPY go.mod go.sum ./

# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go mod tidy

FROM build-common AS dev

# Install air for hot reload & delve for debugging
RUN go install github.com/cosmtrek/air@v1.43.0 && \
    go install github.com/go-delve/delve/cmd/dlv@v1.20.2

# Copy the remaining source code
COPY . .

CMD ["air", "-c", ".air.toml"]

FROM build-common AS build-prod

# Add non root user
RUN useradd -u 1001 nonroot

# Copy all the source code
COPY . .

# Compile healthcheck
RUN go build \
    -ldflags="-linkmode external -extldflags -static" \
    -tags netgo \
    -o healthcheck \
    ./healthcheck/healthcheck.go

# Compile application during build rather than at runtime
# Add flags to statically link binary
RUN go build \
    -ldflags="-linkmode external -extldflags -static" \
    -tags netgo \
    -o api-golang

# Create a minimal docker image with only the binary built in the previous image.
FROM scratch AS prod

# Set gin mode
ENV GIN_MODE=release

# Specify working directory for the app in the docker container
WORKDIR /

# Copy the passwd file
COPY --from=build-prod /etc/passwd /etc/passwd

# Copy the healthcheck binary from the build stage
COPY --from=build-prod /app/healthcheck/healthcheck healthcheck

# Copy the binary from the build stage
COPY --from=build-prod /app/api-golang api-golang

# Use nonroot user
USER nonroot

# Indicate expected port
EXPOSE 8080

# Execute the app
CMD ["/api-golang"]