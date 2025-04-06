### **What Are Environment Variables in Docker?**

Environment variables are key-value pairs used to configure applications within Docker containers. They allow you to set dynamic values that your application can access at runtime, making your applications more flexible and portable.

**Benefits:**

- **Configuration Flexibility**: Easily adjust application settings without modifying code.
- **Security**: Safely pass sensitive information (like API keys or passwords) without hardcoding them.
- **Portability**: Run your application consistently across different environments by changing only the environment variables.

### **Setting Environment Variables in Docker**

1. **In a Dockerfile**:
   Use the `ENV` instruction to define environment variables that will be available in all containers created from the image.

   ```dockerfile
   ENV DB_HOST=database.example.com
   ENV DB_PORT=5432
   ```

2. **At Runtime with `docker run`**:
   Set environment variables when starting a container using the `-e` flag.

   ```bash
   docker run -e DB_HOST=database.example.com -e DB_PORT=5432 my_image
   ```

3. **Using a `.env` File**:
   Store environment variables in a `.env` file and load them into your container.

   **.env file:**

   ```
   DB_HOST=database.example.com
   DB_PORT=5432
   ```

   **Run the container:**

   ```bash
   docker run --env-file .env my_image
   ```

**Note**: Environment variables set at runtime (using `docker run -e`) override those defined in the Dockerfile.

### **Accessing Environment Variables in Your Application**

Within your application code, you can access these environment variables using standard methods provided by your programming language. For example, in Python:

```python
import os

db_host = os.getenv('DB_HOST')
db_port = os.getenv('DB_PORT')
```

This approach ensures that your application can adapt to different environments without requiring code changes.

For more detailed information, you can refer to the following resources:

- [Docker Environment Variables: Setup and Configuration for Server Apps](https://stackify.com/docker-environment-variables/)
- [How to Set Docker Environment Variables](https://phoenixnap.com/kb/docker-environment-variables)

### **Scenario:**
You have a Node.js app that needs to connect to a PostgreSQL database. The database connection details like host, port, and username should be passed as environment variables to the Docker container.

### **1. Dockerfile**

```Dockerfile
FROM node:14

# Set the working directory inside the container
WORKDIR /app

# Copy the application code to the container
COPY . .

# Install dependencies
RUN npm install

# Define default environment variables
ENV DB_HOST=db.example.com
ENV DB_PORT=5432
ENV DB_USER=user
ENV DB_PASSWORD=password

# Start the app
CMD ["node", "app.js"]
```

### **2. Node.js Application (app.js)**

```js
const { Client } = require('pg'); // PostgreSQL client

// Access environment variables
const dbHost = process.env.DB_HOST;
const dbPort = process.env.DB_PORT;
const dbUser = process.env.DB_USER;
const dbPassword = process.env.DB_PASSWORD;

// Setup the database client
const client = new Client({
  host: dbHost,
  port: dbPort,
  user: dbUser,
  password: dbPassword,
  database: 'mydatabase',
});

// Connect to the database
client.connect()
  .then(() => console.log('Connected to the database'))
  .catch(err => console.error('Database connection error', err.stack));
```

### **3. Docker Build and Run**

- **Build the Docker image**:
  ```bash
  docker build -t my-node-app .
  ```

- **Run the Docker container with custom environment variables**:
  ```bash
  docker run -e DB_HOST=custom-db.example.com -e DB_PORT=5433 -e DB_USER=myuser -e DB_PASSWORD=mysecretpassword my-node-app
  ```

### **Explanation:**
- The **Dockerfile** sets up a Node.js application and includes environment variables (`DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`) for the database connection.
- The **Node.js app** (`app.js`) uses the `pg` module to connect to a PostgreSQL database, and the connection details (host, port, username, and password) are provided through environment variables.
- When running the container, the **custom environment variables** can be passed using `-e` flags to override the defaults in the Dockerfile.

### **Key Points**:
- Environment variables are used to keep sensitive information (like DB credentials) out of the code.
- You can configure the app differently in different environments (e.g., staging vs. production) just by changing the environment variables at runtime.
