# genjutsudev-docker

> sudo make php-cli
    
    mysql -u USERNAME -pPASSWORD -h HOSTNAME_OR_IP DATABASE_NAME
    redis-cli -h HOSTNAME_OR_IP -a PASSWORD

# docker-compose.yml

```yml
services:

  nginx:
    container_name: genjutsudev_nginx
    build:
      context: ./genjutsudev-docker/nginx
      dockerfile: nginx.docker
    volumes:
      - ./genjutsudev:/var/www/genjutsudev
      - ./genjutsudev-docker/nginx/sites-available:/etc/nginx/conf.d
    depends_on:
      - php-fpm
    ports:
      - '9000:9000' # genjutsudev
    networks:
      - genjutsudev

  php-fpm:
    container_name: genjutsudev_php-fpm
    build:
      context: ./genjutsudev-docker
      dockerfile: php/fpm.docker
    volumes:
      - ./genjutsudev:/var/www/genjutsudev
    environment:
      XDEBUG_CONFIG: 'remote_host=host.docker.internal remote_enable=1'
      PHP_IDE_CONFIG: 'serverName=Docker'
    networks:
      - genjutsudev

  php-cli:
    container_name: genjutsudev_php-cli
    build:
      context: ./genjutsudev-docker
      dockerfile: php/cli.docker
    volumes:
      - ./genjutsudev:/var/www/genjutsudev
      - genjutsudev_composer:/root/.composer/cache
    depends_on:
      - mysql
      - redis
    networks:
      - genjutsudev

  mysql:
    container_name: genjutsudev_mysql
    build:
      context: ./genjutsudev-docker
      dockerfile: database/mysql.docker
    volumes:
      - genjutsudev_mysql:/var/www/lib/mysql
    environment:
      - MYSQL_DATABASE=genjutsudev
      - MYSQL_ROOT_USER=root
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_ROOT_HOST=%
    ports:
      - '3306:3306'
    networks:
      - genjutsudev

  redis:
    container_name: genjutsudev_redis
    build:
      context: ./genjutsudev-docker
      dockerfile: database/redis.docker
    volumes:
      - genjutsudev_redis:/data
    command:
      - 'redis-server'
      - '--databases 2'
      - '--save 900 1'
      - '--save 300 10'
      - '--save 60 10000'
      - '--requirepass secret'
    networks:
      - genjutsudev

  node:
    container_name: genjutsudev_node
    build:
      context: ./genjutsudev-docker
      dockerfile: node/node.docker
    volumes:
      - ./genjutsudev:/var/www/genjutsudev
    networks:
      - genjutsudev

  mailer:
    image: mailhog/mailhog
    ports:
      - "8085:8025"
    networks:
      - genjutsudev

volumes:
  genjutsudev_composer:
  genjutsudev_mysql:
  genjutsudev_redis:

networks:
  genjutsudev:
```

# Makefile

```Makefile
init: build
up: docker-up
stop: docker-stop
down: docker-down
restart: down up
build: docker-down-clear docker-build docker-up

php-cli:
	docker compose run --rm php-cli bash

php-fpm:
	docker compose run --rm php-fpm bash

docker-up:
	docker compose up -d --build

docker-stop:
	docker compose stop

docker-down:
	docker compose down --remove-orphans

docker-down-clear:
	docker compose down -v --remove-orphans

docker-build:
	docker compose build --pull
```
