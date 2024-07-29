# Laravel desde Docker

## Requisitos previos

### Windows
- Como primer paso, si estás trabajando en Windows, activa WSL, desde donde levantarás una instancia de Ubuntu. 
- Instala Docker en windows y configura para que se conecte con WSL, o instala docker en tu instancia de Ubuntu

## Desde ubuntu, crear la carpeta que será el espacio de trabajo de tu proyecto
```bash
mkdir prueba
```

## Ubicate en la carpeta desde donde vas a trabajar
```bash
cd prueba
```

## Inicia tu proyecto laravel, para este caso mi proyecto se llamará jd
```bash
curl -s https://laravel.build/jd | bash
```
Esto, va a crear una carpeta llamada "jd", la cual contendrá la estructura del proyecto. 

## Para personalizar el puerto desde el cual estará expuesto el proyecto, agregar al archivo .env lo siguiente
```bash
APP_PORT=8080
```

## Para personalizar la base de datos y usar mariaDB en lugar de MySQL, configurar el docker-compose.yml
```bash
version: '3.8'

services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.3
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.3/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
        # Exponer el el puerto 8080, se supone que lo herera de las variables de entorno
            - '${APP_PORT:-8080}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
            IGNITION_LOCAL_SITES_PATH: '${PWD}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
            - redis
            - meilisearch
            - mailpit
            - selenium

    mysql:
        image: 'mariadb:10.5'  # Cambiar a la imagen de MariaDB
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ROOT_HOST: '%'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 1
        volumes:
            - 'sail-mysql:/var/lib/mysql'
            - './vendor/laravel/sail/database/mysql/create-testing-database.sh:/docker-entrypoint-initdb.d/10-create-testing-database.sh'
        networks:
            - sail
        healthcheck:
            test:
                - CMD
                - mysqladmin
                - ping
                - '-p${DB_PASSWORD}'
            retries: 3
            timeout: 5s

    redis:
        image: 'redis:alpine'
        ports:
            - '${FORWARD_REDIS_PORT:-6379}:6379'
        volumes:
            - 'sail-redis:/data'
        networks:
            - sail
        healthcheck:
            test:
                - CMD
                - redis-cli
                - ping
            retries: 3
            timeout: 5s

    meilisearch:
        image: 'getmeili/meilisearch:latest'
        ports:
            - '${FORWARD_MEILISEARCH_PORT:-7700}:7700'
        environment:
            MEILI_NO_ANALYTICS: '${MEILISEARCH_NO_ANALYTICS:-false}'
        volumes:
            - 'sail-meilisearch:/meili_data'
        networks:
            - sail
        healthcheck:
            test:
                - CMD
                - wget
                - '--no-verbose'
                - '--spider'
                - 'http://127.0.0.1:7700/health'
            retries: 3
            timeout: 5s

    mailpit:
        image: 'axllent/mailpit:latest'
        ports:
            - '${FORWARD_MAILPIT_PORT:-1025}:1025'
            - '${FORWARD_MAILPIT_DASHBOARD_PORT:-8025}:8025'
        networks:
            - sail

    selenium:
        image: selenium/standalone-chrome
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        volumes:
            - '/dev/shm:/dev/shm'
        networks:
            - sail

networks:
    sail:
        driver: bridge

volumes:
    sail-mysql:
        driver: local
    sail-redis:
        driver: local
    sail-meilisearch:
        driver: local
```

## Para Crear los contenedores y dejarlos corriendo en segundo plano
```bash
./vendor/bin/sail up -d --build
```

## Para correr el migrate
```bash
./vendor/bin/sail artisan migrate
```

## Para trabajar con tu proyecto desde visual Studio Code
```bash
code .
```

## Para detener el contenedor
```bash
./vendor/bin/sail down -v
```

## Verifica el proyecto desde el navegador
```bash
localhost:8080
```