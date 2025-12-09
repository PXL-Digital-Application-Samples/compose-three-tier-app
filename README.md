# Exercise: Containerize a Three-Tier Application

<https://github.com/PXL-Digital-Application-Samples/compose-three-tier-app>

You have been provided with the source code for a classic Three-Tier application. Your task is to containerize this application by creating a `Dockerfile` for each layer and orchestrating them together using a `compose.yaml` file.

**The Application Architecture:**

- **Frontend:** React.js application (Needs to be built and served statically).
- **Backend:** Node.js application (API).
- **Database:** MySQL (Relational Data Store).

---

## Tasks

### Task A: The Database (MySQL)

Create a `Dockerfile` inside the `mysql` directory.

**Requirements:**

- **Base Image:** Use version **8.0** of the official MySQL image.
- **Authentication:** Configure the environment variables to set the root password to `mysql123` and create a database named `school`.
- **Legacy Compatibility:** Since our Backend uses an older Node.js version, you must configure the MySQL server to use the native password authentication plugin.
  - *Hint:* Override the default command with: `mysqld --default-authentication-plugin=mysql_native_password`
- **Persistence:** Ensure the database data is stored in a volume at `/var/lib/mysql`.

### Task B: The Backend (Node.js)

Create a `Dockerfile` inside the `backend` directory.

**Requirements:**

- **Base Image:** Use Node.js version **14**.
- **Work Directory:** Set the working directory inside the container to `/app`.
- **Dependencies:** Copy package files and install dependencies (`npm install`).
- **Source:** Copy the remaining application code.
- **Networking:** Expose port **3500**.
- **Execution:** Start the app with `node server.js`.

**Note:** This application expects specific environment variables to connect to the database. You will configure the *values* for these in Task D (Docker Compose), but the keys expected by the code are:

- `host`
- `user`
- `password`
- `database`

### Task C: The Frontend (Multi-Stage Build)

Create a `Dockerfile` inside the `frontend` directory. This **must** be a multi-stage build.

**Requirements:**

- **Stage 1 (Build Stage):**
  - Use Node.js version **21 (Alpine variant)**.
  - Copy source files and install dependencies.
  - **API Configuration:** The React code expects an environment variable named `REACT_APP_API_BASE_URL`.
    - You must set this variable to `http://localhost:3500` **before** running the build command.
    - *Why Localhost?* Because the React code runs in the User's Browser, not inside the Docker container.
  - Run the build script (`npm run build`) to generate the static files.
- **Stage 2 (Production Stage):**
  - Use **Nginx (Alpine variant)**.
  - Copy the *build artifacts* from Stage 1 into the Nginx html directory (`/usr/share/nginx/html`).
  - Expose port **80** and start Nginx.

### Task D: Orchestration (Docker Compose)

Create a `compose.yaml` file in the root directory.

**Requirements:**

- **Services:** Define `frontend`, `backend`, and `mysql-db`.
- **Port Mapping:**
  - Frontend: Host **80** -\> Container **80**.
  - Backend: Host **3500** -\> Container **3500**.
  - Database: Host **3306** -\> Container **3306**.
- **Networking:** Put all services on a custom bridge network named `three-tier-network`.
- **Environment Configuration (Backend):**
  - Configure the backend service with the environment variables listed in Task B.
  - **Important:** The `host` variable must match the **container name** or **service name** of your database container.
- **Volumes:** Mount a named volume to the database service for persistence.
- **Dependencies:** Use `depends_on` to ensure the backend starts after the database.

---

## Verification Steps

1. **Build and Run:**

    ```bash
    docker compose up --build -d
    ```

2. **Initialize Database:**
    Open a shell inside the database container and create the required table:

    ```bash
    docker compose exec mysql-db mysql -u root -p
    # Password: mysql123
    ```

    *(Paste the following SQL)*:

    ```sql
    USE school;
    CREATE TABLE student (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(40), roll_number INT, class VARCHAR(16));
    CREATE TABLE teacher (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(40), subject VARCHAR(40), class VARCHAR(16));
    exit;
    ```

3. **Final Test:**
    Open `http://localhost` in your browser.

      - Click "Teachers".
      - Add a new teacher.
      - If the data saves and appears in the list, your Full Stack application is working\!

---

## Common Pitfalls & Hints

> 1. **Frontend Variables:** React Environment variables (starting with `REACT_APP_`) are "baked in" at build time. If you change the variable in the Dockerfile, you must run `docker compose build --no-cache frontend` to see the change.
> 2. **Backend Connection:** If the backend crashes with `ECONNREFUSED 127.0.0.1:3306`, it means the backend is trying to connect to *itself* (localhost). Ensure the `host` environment variable is set to the **Database Service Name** defined in your compose file.
> 3. **Database Version:** We use MySQL 8.0 with the `--default-authentication-plugin=mysql_native_password` flag because the Node.js 14 MySQL driver is too old to understand the newer default MySQL authentication.
