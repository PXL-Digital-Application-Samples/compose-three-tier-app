# Exercise: Containerize a Three-Tier Application

You have been provided with the source code for a classic Three-Tier application (Client, Server, Database). Your task is to containerize this application by creating a `Dockerfile` for each layer and orchestrating them together using a `compose.yaml` file.

**The Application Architecture:**

- **Frontend:** React.js application (Needs to be built and served statically).
- **Backend:** Node.js application (API).
- **Database:** MySQL (Relational Data Store).

---

## Tasks

### Task A: The Database (MySQL)

Create a `Dockerfile` inside the `mysql` directory.

**Requirements:**

- **Base Image:** Use version 8.0 of the official MySQL image.
- **Authentication:** You must configure the environment variables to set the root password to `mysql123`.
- **Initialization:** Configure the container to automatically create a database named `school` upon startup.
- **Networking:** The database listens on the standard MySQL port.
- **Persistence:** Ensure the database files are stored in a volume so data is not lost if the container restarts.

### Task B: The Backend (Node.js)

Create a `Dockerfile` inside the `backend` directory.

**Requirements:**

- **Base Image:** Use Node.js version **14**.
- **Work Directory:** Set the working directory inside the container to `/app`.
- **Dependencies:** Copy the package files and install the dependencies using `npm install`.
- **Source:** Copy the remaining application code.
- **Networking:** The application runs on port **3500**. Make sure this port is exposed.
- **Execution:** The app should start by running `server.js`.

### Task C: The Frontend (Multi-Stage Build)

Create a `Dockerfile` inside the `frontend` directory. This **must** be a multi-stage build to optimize image size.

**Requirements:**

- **Stage 1 (Build Stage):**
  - Use Node.js version **21 (Alpine variant)**.
  - Copy source files and install dependencies.
  - Run the build script (`npm run build`) to generate the static files.
- **Stage 2 (Production Stage):**
  - Use **Nginx (Alpine variant)** as the base.
  - Copy the *build artifacts* (the static files generated in Stage 1) into the Nginx html directory (`/usr/share/nginx/html`).
  - ***Networking:** Expose port **80**.
  - **Execution:** Start Nginx in the foreground.

### Task D: Orchestration (Docker Compose)

Create a `compose.yaml` file in the root directory to run all three services together.

**Requirements:**

- **Services:** Define three services: `frontend`, `backend`, and `mysql-db`.
- **Build Contexts:** Each service should build from its respective directory.
- **Port Mapping:**
  - Frontend: Map host port **80** to container port **80**.
  - Backend: Map host port **3500** to container port **3500**.
  - Database: Map host port **3306** to container port **3306**.
- **Networking:** All services must be on the same custom bridge network (e.g., `three-tier-network`) to allow them to communicate by service name.
- **Volumes:** Define a named volume (e.g., `mysql-data`) and mount it to `/var/lib/mysql` in the database service to ensure data persistence.
- **Dependency Management:** Ensure the `backend` service does not attempt to start until the `mysql-db` service is running (Hint: use `depends_on`).

---

## Verification Steps

Once you have created the files, run the application using:

```bash
docker compose up --build -d
```

To pass the exercise, you can successfully execute the following steps:

- **Database Setup:** Access the running database container and verify the `school` database exists.

  ```bash
  docker compose exec mysql-db mysql -u root -p
  # Enter password: mysql123
  SHOW DATABASES;
  ```

- **Table Creation:** Inside the MySQL shell, manually run the following SQL to ensure the DB is writable:

  ```sql
  USE school;
  CREATE TABLE student (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(40), roll_number INT, class VARCHAR(16));
  ```

- **Browser Test:** Navigate to `http://localhost` in your browser. You should see the React application running.

---

## Helpful Hints

> - **Frontend:** In the multi-stage build, remember that the build artifacts usually land in a folder named `/app/build` or `/app/dist` in the first stage.
> - **Backend Connection:** If the backend fails to connect to the database, check that your `compose.yaml` service names match what the backend expects (or ensure they are on the same network).
