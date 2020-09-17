---
title: Config docker cho Rails App nhanh gọn lẹ
date: 2020-09-17 10:56:53
tags:
  - Docker
  - Rails
  - Technical
---

> **Note tạm vào đây từ từ viết chi tiết sau**

`Dockerfile`

```bash
FROM ruby:2.7.1

RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - \
  && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
  && apt-key update \
  && apt-get update -qq \
  && apt-get install -y --no-install-recommends build-essential libpq-dev nodejs yarn less \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

WORKDIR /app

COPY Gemfile* ./

CMD ["/app/docker_bash.sh"]
```

`docker-compose.yml`
```yml
version: "3"
services:
  api:
    build: .
    volumes:
      - .:/app
      - bundle:/usr/local/bundle
    ports:
      - 3000:3000
    depends_on:
      - db
    tty: true
    stdin_open: true

  db:
    image: mysql:5.7
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root

volumes:
  mysql_data:
  bundle:
```

`docker_bash.sh`
```bash
#!/bin/sh
rm -f tmp/pids/server.pid
bundle install -j4 --retry 5
bundle exec rails db:prepare
bundle exec rails s -b '0.0.0.0'
```
