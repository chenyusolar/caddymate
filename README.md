为了在 Docker Compose 中支持多个版本的 PHP，你可以通过定义多个 PHP 服务，并使用不同的 PHP 镜像来启动不同版本的 PHP。通过使用 `docker-compose.override.yml` 文件或环境变量，你可以在启动时选择要运行的 PHP 版本。

### 1. **使用环境变量选择 PHP 版本**

你可以通过环境变量来动态选择 PHP 版本。以下是一个示例 `docker-compose.yml` 文件，其中可以通过环境变量来启动不同版本的 PHP。

#### `docker-compose.yml`

```yaml
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - php
    networks:
      - backend

  php:
    image: php:${PHP_VERSION}-fpm    # 使用环境变量选择PHP版本
    volumes:
      - ./site:/srv
    networks:
      - backend

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend

volumes:
  caddy_data:
  caddy_config:
  mysql_data:

networks:
  backend:
    driver: bridge
```

#### 说明：
- **`php:${PHP_VERSION}-fpm`**：这里使用了 Docker 的动态镜像标签功能。`${PHP_VERSION}` 是一个环境变量，启动时可以指定要使用的 PHP 版本。
- Docker Hub 上提供了多个 PHP 版本的镜像标签，比如 `php:7.0-fpm`、`php:7.4-fpm`、`php:8.3-fpm`，因此你可以灵活选择需要的版本。

### 2. **通过环境变量选择 PHP 版本**

你可以通过以下方式设置 `PHP_VERSION` 环境变量，从而选择 PHP 版本。

#### 方法 1：在 `.env` 文件中设置

创建 `.env` 文件，并在其中定义 PHP 版本：

```env
PHP_VERSION=7.4
```

然后在项目目录中使用 `docker-compose up` 启动时，Docker Compose 会自动读取 `.env` 文件，选择 PHP 7.4 版本。

#### 方法 2：在命令行中设置环境变量

你也可以通过命令行直接设置 `PHP_VERSION`：

```bash
PHP_VERSION=8.3 docker-compose up -d
```

这样会在运行时使用 PHP 8.3 版本。

### 3. **定义多个 PHP 服务并选择版本**

你可以在 `docker-compose.yml` 中定义多个 PHP 版本的服务，并通过 `docker-compose.override.yml` 文件或直接使用特定服务来选择需要的 PHP 版本。

#### 示例：定义多个 PHP 服务

```yaml
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - php7
    networks:
      - backend

  php7:
    image: php:7.4-fpm
    volumes:
      - ./site:/srv
    networks:
      - backend

  php8:
    image: php:8.0-fpm
    volumes:
      - ./site:/srv
    networks:
      - backend

  php83:
    image: php:8.3-fpm
    volumes:
      - ./site:/srv
    networks:
      - backend

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend

volumes:
  caddy_data:
  caddy_config:
  mysql_data:

networks:
  backend:
    driver: bridge
```

#### 说明：
- 定义了多个 PHP 版本的服务（`php7`、`php8` 和 `php83`），每个服务使用不同的 PHP 版本。
- 在 `caddy` 服务的 `depends_on` 中指定需要的 PHP 版本。例如，如果你要使用 PHP 7.4，你可以设置 `depends_on` 为 `php7`，如果想要切换到 PHP 8.3，只需将 `depends_on` 修改为 `php83`。

#### 启动指定的 PHP 服务：

```bash
docker-compose up -d caddy php83
```

这个命令会启动 Caddy 和 PHP 8.3 服务。

### 总结：
- 使用 Docker Compose 的环境变量功能，可以灵活选择不同的 PHP 版本。
- 通过 `.env` 文件或命令行变量可以动态设置 PHP 版本。
- 你也可以通过定义多个 PHP 服务并在启动时选择需要的服务来实现多个版本的 PHP 切换。

这种方式使你能够轻松在开发或测试环境中切换不同的 PHP 版本。