# 🐳 Understanding Docker’s Embedded DNS with .NET + MySQL Example

Docker provides a **built-in (embedded) DNS server** to make service discovery easy inside container networks. This DNS system allows containers to resolve each other by name (i.e., the container name), avoiding the need for hardcoded IP addresses.

This project demonstrates that with a simple **.NET 6 Web API** that connects to a **MySQL** database container using the **container name as the hostname**—thanks to Docker's internal DNS.

---

## 🧠 What is Docker's Embedded DNS?

Docker's embedded DNS server allows containers on the **same custom user-defined bridge network** to resolve each other by **container name**. Here's how it works:

- Docker runs an internal DNS server at `127.0.0.11` (visible from within containers).
- If two containers are on the same custom network, one can ping the other using its container name.
- This is safer and more robust than using container IPs, which can change.

---

## 🛠 Example: .NET WebApp Resolving MySQL by Name

This example demonstrates:

- A `.NET 6` Web API that connects to MySQL using `mysql-db` as the hostname.
- A custom Docker bridge network created automatically via Docker Compose.
- Docker's embedded DNS resolving `mysql-db` to the correct IP dynamically.

---

## 📁 Project Structure

```

docker-dns-dotnet-example/
│
├── docker-compose.yml
├── WebApp/
│   ├── Program.cs
│   ├── WebApp.csproj
│   └── Dockerfile
└── README.md

````

---

## 🔧 .NET WebApp Code (`WebApp/Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", async () =>
{
    string connectionString = "Server=mysql-db;Database=testdb;User=root;Password=example;";
    using var connection = new MySql.Data.MySqlClient.MySqlConnection(connectionString);
    await connection.OpenAsync();

    var cmd = connection.CreateCommand();
    cmd.CommandText = "SELECT NOW();";
    var result = await cmd.ExecuteScalarAsync();

    return $"Database time from MySQL: {result}";
});

app.Run();
````

✅ Here, `mysql-db` is the **name of the MySQL container** — not an IP. Docker's embedded DNS resolves this name to the internal IP of the container at runtime.

---

## 🐋 Dockerfile for WebApp (`WebApp/Dockerfile`)

```dockerfile
# Build Stage
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

# Runtime Stage
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "WebApp.dll"]
```

---

## 📦 Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'

services:
  mysql-db:
    image: mysql:8
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: testdb
    networks:
      - internal-net

  webapp:
    build:
      context: ./WebApp
    container_name: webapp
    depends_on:
      - mysql-db
    ports:
      - "5000:80"
    networks:
      - internal-net

networks:
  internal-net:
    driver: bridge
```

Docker Compose automatically creates the **`internal-net`** bridge network. Both containers are attached to this network, allowing Docker's DNS to resolve their names internally.

---

## 🚀 Running the Example

1. Clone or copy the project.
2. Open a terminal and run:

```bash
docker-compose up --build
```

3. Open your browser and go to:
   [http://localhost:5000](http://localhost:5000)

You’ll see something like:

```
Database time from MySQL: 2025-05-04 10:34:01
```

---

## 🧪 Test the Embedded DNS Manually

Open a terminal in the `webapp` container:

```bash
docker exec -it webapp ping mysql-db
```

Expected output:

```
PING mysql-db (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: icmp_seq=0 ttl=64 time=0.087 ms
```

This shows that `mysql-db` is being **resolved via Docker’s internal DNS** to the correct container IP.

---

## 🔍 How to Inspect the DNS Resolver Inside a Container

You can also check the DNS resolver IP inside the container:

```bash
docker exec -it webapp cat /etc/resolv.conf
```

Output includes:

```
nameserver 127.0.0.11
```

This proves that Docker's embedded DNS is configured in the container.

---

## 🔄 Why Not Use IPs?

Using hardcoded IPs like `172.20.0.2` is a bad idea:

* IPs change if containers are recreated.
* DNS is dynamic, reliable, and consistent with container naming.

---

## 🧹 Clean Up

```bash
docker-compose down
```

This stops and removes containers, network, and volumes (if any).

---

## ✅ Summary

| Feature               | Benefit                                                  |
| --------------------- | -------------------------------------------------------- |
| Embedded DNS          | Docker handles name resolution internally (`127.0.0.11`) |
| Container name usage  | Safer than IPs for service discovery                     |
| Custom bridge network | Enables DNS between containers                           |
| Real use case         | .NET API resolves MySQL hostname to connect              |

---

## 📚 Further Reading

* [Docker Networking Docs](https://docs.docker.com/network/)
* [Docker DNS Resolution](https://docs.docker.com/network/network-tutorial-standalone/#use-docker-networking)
* [.NET MySQL Connector](https://www.nuget.org/packages/MySql.Data)

