# Joplin-Server with Docker Compose

This guide will help you set up Joplin-Server using Docker Compose. Follow the steps below to configure your environment and get Joplin-Server running with an external PostgreSQL database.

## Prerequisites

- Docker installed on your system.
- Docker Compose installed on your system.
- NTP Server installed on your system.
- Postgres installed on your system.
- pgAdmin installed for database management.

## Docker Compose Setup

Create a `docker-compose.yml` file with the following content:

```yaml
services:
  joplin-server:
    image: etechonomy/joplin-server:latest
    container_name: joplin-server
    environment:
      - APP_BASE_URL=http://localhost.local
      - APP_PORT=22300
      - APP_NAME=Joplin-Server
      - NTP_SERVER=${NTP_SERVER_IP}  # Add Local NTP Server as it seems Joplin won't boot correctly with no internet connection
      - MAX_TIME_DRIFT=5000
      - DB_CLIENT=pg
      - NODE_ENV=production
      - MAILER_HOST=smtp.gmail.com
      - MAILER_PORT=587
      - MAILER_SECURITY=starttls
      - MAILER_AUTH_USER=${MAILER_AUTH_USER}
      - MAILER_ENABLED=1
      - MAILER_NOREPLY_NAME=joplin server
      - MAILER_NOREPLY_EMAIL=${MAILER_NOREPLY_EMAIL}
      - MAILER_AUTH_PASSWORD=${MAILER_AUTH_PASSWORD}
      - POSTGRES_CONNECTION_STRING=postgresql://joplin:${DB_PASSWORD}@${POSTGRES_IP}:5432/joplin
    restart: unless-stopped
    ports:
      - 22300:22300
    networks:
      - postgres_network

networks:
  postgres_network:
    name: postgres_network
    driver: bridge
```
Ensure that you change the IP address for the NTP Server and Postgres Database.

### Environment Variables

Create a `.env` file in the same directory as your `docker-compose.yml` file with the following content:

```dotenv
# Email configuration for mailer service
MAILER_AUTH_USER=changeme@user.com  # Username for SMTP authentication
MAILER_NOREPLY_EMAIL=changeme@user.com  # Email address for no-reply emails
MAILER_AUTH_PASSWORD=Your_Password  # Password for SMTP authentication

# Database configuration
DB_PASSWORD=Echo45Admin*  # Password for the PostgreSQL database user

# NTP server configuration
NTP_SERVER_IP=10.0.0.10  # IP address of the NTP server for time synchronization

# PostgreSQL database configuration
POSTGRES_IP=10.0.0.202  # IP address of the PostgreSQL server
```


## Database Setup

Before running the Docker Compose setup, you need to create the database and user using pgAdmin.

1. **Open pgAdmin** and connect to your PostgreSQL server.
2. **Create a new database**:
   - Name: `joplin`
3. **Create a new user**:
   - Username: `joplin`
   - Password: `your_password` (match this with `DB_PASSWORD` in your `.env` file)
4. **Set user privileges**:
   - Ensure the `joplin` user has `Can Login` permissions.
   - Assign the `joplin` user as the owner of the `joplin` database.


## Running the Joplin-Server

After setting up the database, you can start the Joplin-Server using Docker Compose.

1. Navigate to the directory containing your `docker-compose.yml` and `.env` files.
2. Run the following command to start the services:
```sh
   docker-compose up -d
```

## Accessing Joplin-Server

Once the services are up and running, you can access the Joplin-Server from your web browser.

- Open your browser and go to the URL specified in `APP_BASE_URL` (e.g., `http://localhost:22300`).

### Initial Login

- Default email: `admin@localhost`
- Default password: `admin`

### Changing Default User Details

1. Log in with the default credentials.
2. Navigate to `Admin Menu -> Users -> admin (user)`.
3. Update the default user, email, and password.
4. Click `Update Profile` when done.

## Notes

- Ensure that both Joplin-Server and PostgreSQL are on the same Docker network (`postgres_network`) for them to communicate properly.
- The `POSTGRES_CONNECTION_STRING` is used to connect Joplin-Server to the PostgreSQL database directly. Adjust the IP and port according to your PostgreSQL setup.

