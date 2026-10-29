# Library gRPC Microservice
 
A Python gRPC service for running a digital library. Books live in **MySQL**, patrons live in **MongoDB**, and everything ships in Docker with migrations that run on their own at startup.
 
---
 
## What It Does
 
| Capability | Description |
|---|---|
| **Books** | Add and look up books by title, author, or ISBN, and track copies in stock |
| **Patrons** | Register patrons and see which books they currently have out |
| **Checkout / Return** | Borrow and return books, with validation (e.g. no checkout when zero copies remain) |
| **Inventory Summary** | Total titles, available copies, and checked-out copies at a glance |
 
## Why Two Databases?
 
The data has two different shapes, so each gets the store that suits it:
 
- **MySQL** holds books: structured, relational, and constrained (unique ISBNs, inventory counts).
- **MongoDB** holds patrons: flexible documents that embed each patron's borrowed-book list.
## Tech Stack
 
- **Language:** Python 3.11
- **API:** gRPC with Protocol Buffers
- **Databases:** MySQL 8.0, MongoDB 6
- **Infrastructure:** Docker and Docker Compose
- **Migrations:** Applied automatically when the service starts
## Architecture
 
The code follows clean architecture, so business rules never depend on a specific database or transport:
 
```
gRPC handlers  →  services (business logic)  →  repositories  →  MySQL / MongoDB
```
 
- **Handlers** translate gRPC requests and responses.
- **Services** enforce the rules (availability checks, borrow limits).
- **Repositories** hide storage details, so a database can be swapped without touching the logic.
## Getting Started
 
**Prerequisites:** Docker and Docker Compose.
 
```bash
# Clone and enter the project
git clone <your-repo-url>
cd <your-repo-name>
 
# Build and start the service with both databases
docker compose up --build
```
 
On first launch the service waits for MySQL and MongoDB, applies migrations, and then begins serving gRPC requests.
 
## Trying It Out
 
With [grpcurl](https://github.com/fullstorydev/grpcurl) installed, you can explore the API directly:
 
```bash
# List available services
grpcurl -plaintext localhost:50051 list
 
# Call an endpoint (method names depend on your .proto definitions)
grpcurl -plaintext -d '{"isbn": "9780143127741"}' \
  localhost:50051 library.LibraryService/GetBook
```
 
> Adjust the port, package, and method names to match your `.proto` files and Compose configuration.
 
## Configuration
 
Settings are provided as environment variables in `docker-compose.yml`, typically:
 
| Variable | Purpose |
|---|---|
| `MYSQL_HOST`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE` | MySQL connection |
| `MONGO_URI` | MongoDB connection string |
| `GRPC_PORT` | Port the gRPC server listens on |
 
## Contributing
 
1. Fork the repo and create a feature branch.
2. Keep new logic in the service layer, with storage code in repositories.
3. Regenerate protobuf stubs after changing any `.proto` file.
4. Open a pull request describing the change.
