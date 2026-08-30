# Objective

Use the MacBook hardware to run a lightweight NAS (Network Attached Storage) server for file sharing and backup purposes.

The MacBook's internal storage will be used as the primary persistent storage for files. The NAS service will be accessible by devices connected to the same local network.

The NAS application will run inside a Docker container, with a directory on the MacBook's internal storage mounted into the container as a persistent volume.

The server will be implemented in Go, exposing a network-accessible interface for managing, uploading, downloading, and backing up files.

# Incremental Tasks
### Phase 1 — Project Setup
 - Set up the Go development environment.
 - Set up Docker and verify that Docker containers can run on the MacBook.
 - Initialize the Go project and module.
 - Define the project directory structure.
 - Create a minimal HTTP server in Go.
 - Create a basic Dockerfile.
 - Build and run the Go application as a Docker container.

### Phase 2 — Storage
 - Define a host directory to act as the NAS storage location.
 - Mount the host directory into the Docker container.
 - Make the storage path configurable through environment variables.
 - Verify that files written by the application persist after the container is stopped/recreated.
 - Implement basic file and directory operations:
 - List files.
 - Create directories.
 - Upload files.
 - Download files.
 - Delete files.
 - Rename/move files.
 - Prevent access outside the configured NAS storage directory.

### Phase 3 — File Sharing API
 - Design a REST API for file operations.
 - Implement directory browsing.
 - Implement multipart file uploads.
 - Implement file downloads.
 - Implement file deletion.
 - Implement directory creation/deletion.
 - Return appropriate HTTP status codes and error messages.
 - Support large files without loading the entire file into memory.

Example API:

GET    /api/files
GET    /api/files/{path}
POST   /api/files/{path}
DELETE /api/files/{path}
POST   /api/directories/{path}
DELETE /api/directories/{path}

### Phase 4 — Web Interface
 - Create a simple web-based NAS interface.
 - Display files and directories.
 - Allow users to navigate directories.
 - Add file upload functionality.
 - Add file download functionality.
 - Add delete and rename functionality.
 - Display file sizes and modification times.
 - Make the interface usable from desktop and mobile devices on the LAN.

### Phase 5 — Authentication & Security
 - Add user authentication.
 - Protect file-management endpoints.
 - Store passwords securely using password hashing.
 - Implement session/token-based authentication.
 - Prevent path traversal attacks.
 - Restrict the service to the local network by default.
 - Validate uploaded filenames and paths.
 - Set reasonable upload/file-size limits.
 - Ensure the Docker container runs with only the filesystem permissions it requires.

### Phase 6 — Backup
 - Define the backup requirements.
 - Implement a backup endpoint or backup operation.
 - Support copying selected files/directories to a backup location.
 - Add scheduled backups.
 - Provide backup status and error reporting.
 - Avoid corrupting or partially overwriting existing backups.
 - Consider incremental backups to reduce unnecessary disk usage.

Important: storing the NAS data on the MacBook's internal disk is not itself a backup. A real backup should eventually reside on a separate physical disk or another machine.

### Phase 7 — Docker Deployment
 - Create a production Dockerfile.
 - Create a docker-compose.yml.
 - Configure persistent storage using a bind mount.
 - Configure the NAS port through environment variables.
 - Configure the storage directory through environment variables.
 - Add Docker health checks.
 - Configure automatic container restart.
 - Document container startup/shutdown procedures.

Example configuration:

services:
  nas:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - ./nas-data:/data
    environment:
      - NAS_STORAGE_PATH=/data
    restart: unless-stopped

### Phase 8 — Local Network Access
 - Make the Go server listen on the appropriate container interface.
 - Expose the NAS service through Docker.
 - Determine the MacBook's LAN IP address.
 - Verify access from another computer on the same network.
 - Verify file upload/download from another device.
 - Test access from a phone/tablet on the same LAN.
 - Document how users connect to the NAS.

### Phase 9 — Reliability
 - Add structured logging.
 - Handle application/container restarts safely.
 - Handle concurrent file operations.
 - Prevent data corruption from simultaneous writes.
 - Add graceful HTTP server shutdown.
 - Add health/status endpoint.
 - Monitor available disk space.
 - Return a clear error when storage is unavailable or full.

### Phase 10 — Testing
 - Unit-test file-management logic.
 - Unit-test path validation.
 - Test upload/download functionality.
 - Test concurrent file operations.
 - Test authentication.
 - Test Docker volume persistence.
 - Test container restart behavior.
 - Test access from multiple LAN clients.
 - Test large-file transfers.
 - Test behavior when disk space is exhausted.


## Technical Requirements

- Go — NAS server implementation.
- Docker — application containerization.
- MacBook — host machine and primary storage.
- HTTP/REST — network API for file operations.
- Storage
- MacBook internal storage will be used as persistent NAS storage.
- The host storage directory must be mounted into the Docker container.
- Container-local storage must not be relied upon for persistent data.
- Networking
- The NAS service must be accessible from devices on the same local network.
- The server should listen on a configurable port.
- LAN access should be supported without requiring an external cloud service.
- Security
- Authentication should be supported before exposing the service beyond a trusted local network.
- All filesystem paths must be validated to prevent path traversal.
- The application must only access the configured storage directory.

## Final Deliverable

The completed project should provide:

MacBook
   │
   ├── Internal Storage
   │       └── NAS data
   │
   └── Docker
         │
         └── Go NAS Server
                 │
                 └── HTTP API / Web UI
                         │
              ┌──────────┼──────────┐
              │          │          │
           Laptop      Phone      Desktop
              │          │          │
              └──── Local Network ──┘


The end result is a self-hosted, Dockerized Go NAS server using the MacBook's internal storage for persistent file sharing and backup operations over the local network.