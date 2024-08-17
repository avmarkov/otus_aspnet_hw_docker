### Домашняя работа. Сделать docker compose file + Настроить CI для своего проекта
#### 1. Добавляем docker-compose файл, где конфигурируем PostgreSQL БД
Добавил docker-compose.yml.
У postgres такие порты - 5434:5432, т.к. у меня на машине установлены два сервера БД Postgres? на портах 5432 и 5433.

```yml
version: '3.8'

services:

  web:
    container_name: 'webapi'
    image: 'webapi'
    build:
      context: .
      dockerfile: Dockerfile
    ports:
     - "5000:5000"
    environment:
     - ASPNETCORE_ENVIRONMENT=Development
     - ConnectionStrings__SQLConnectionString=Host=postgres;Port=5432;Database=promocodefactorydb;Username=postgres;Password=password;
    depends_on:
     - "postgres"

  postgres:
    image: postgres:latest
    hostname: postgres
    restart: always
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - pg-data:/var/lib/postgresql/data
    ports:
      - "5434:5432"
      
volumes:
  pg-data:

networks:
  default:
    name: myLocalNetwork # создана извне: docker network create --driver=bridge myLocalNetwork
    external: true 
```   

##### 2. Добавляем начальную миграцию БД
Добавил

##### 3. Вместо SQLite добавляем PostgreSQL в приложение и настраиваем EF на работу с PostgreSQL, проверяем через Swagger, что Read/Create/Update методы работают

В проекте поменял SQLite на PostgreSQL

```cs
services.AddDbContext<DataContext>(options =>
{
    options.UseNpgsql(Environment.GetEnvironmentVariable("ConnectionStrings__SQLConnectionString") ??
                      Configuration.GetConnectionString("SQLConnectionString") ??
                      throw new Exception("No connection string to sql database"));
});
```

Настройки в файле appsettings.json такие:
```json
{
  "ConnectionStrings": {
    "SQLConnectionString": "Host=localhost;Port=5434;Database=promocodefactorydb;Username=postgres;Password=password"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

Все работает:

<image src="images/swagger.png" alt="swagger">

Причем не только пострег развернут в docker, но и само приложение:

<image src="images/docker.png" alt="docker">

#### 4. Настраиваем сборку на Gitlab
Настроил:

<image src="images/ci.png" alt="ci">
