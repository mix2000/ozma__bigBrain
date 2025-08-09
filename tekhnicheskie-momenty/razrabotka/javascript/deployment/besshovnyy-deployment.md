---
order: 1
title: Бесшовный deployment
---

Бесшовный деплой (zero-downtime deployment) для приложения на NestJS можно организовать с помощью таких подходов, как **blue-green deployment**, **rolling deployment** или с использованием процессов, которые поддерживают горячую перезагрузку. Вот пошаговый процесс:

---

### 1\. **Подготовка приложения**

-  **Graceful Shutdown**: Настрой graceful shutdown для корректного завершения работы при обновлении.

   ```
   import { Injectable, OnApplicationShutdown } from '@nestjs/common';
   
   @Injectable()
   export class AppService implements OnApplicationShutdown {
     onApplicationShutdown(signal?: string) {
       console.log(`App shutting down... Signal: ${signal}`);
       // Закрой соединения, очисти ресурсы
     }
   }
   
   ```

   В `main.ts` добавь хук:

   ```
   import { AppModule } from './app.module';
   import { NestFactory } from '@nestjs/core';
   
   async function bootstrap() {
     const app = await NestFactory.create(AppModule);
   
     // Graceful shutdown hooks
     app.enableShutdownHooks();
   
     await app.listen(3000);
   }
   bootstrap();
   
   ```

---

### 2\. **Использование Docker**

Используй контейнеризацию для управления деплоем. Пример `Dockerfile`:

```
# Step 1: Build Stage
FROM node:18 as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Step 2: Run Stage
FROM node:18
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/main.js"]
```

Настрой `docker-compose.yml` для удобства:

```
version: '3.8'
services:
  app:
    image: my-nestjs-app:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    restart: always
```

---

### 3\. **Организация бесшовного деплоя**

-  **Rolling Deployment**:\
   Используй инструменты оркестрации, такие как Kubernetes, чтобы обновлять контейнеры по одному.

-  **Blue-Green Deployment**:\
   Создай две версии приложения (например, `v1` и `v2`), перенаправляя трафик только после того, как новая версия готова.

Пример для NGINX:

```
upstream app {
  server app-v1:3000;
  server app-v2:3000 backup;
}

server {
  listen 80;
  location / {
    proxy_pass http://app;
  }
}
```

Обновляй конфигурацию, когда `v2` станет активной.

---

### 4\. **PM2 для управления процессами**

PM2 поддерживает zero-downtime через перезагрузку с балансировкой процессов.

Установка:

```
npm install pm2 -g
```

Запуск с PM2:

```
pm2 start dist/main.js --name my-nestjs-app
```

Перезапуск без простоя:

```
pm2 reload my-nestjs-app
```

---

### 5\. **CI/CD**

Интеграция в CI/CD, например, GitHub Actions:

```
name: Deploy NestJS App

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Build application
        run: npm run build

      - name: Deploy to Server
        run: |
          scp -r dist/ user@your-server:/path/to/app
          ssh user@your-server 'pm2 reload my-nestjs-app'
```

---

### 6\. **Тестирование перед деплоем**

Убедись, что новая версия приложения протестирована:

-  **e2e тесты**: Используй `@nestjs/testing`.

-  **Health Check**: Реализуй endpoint для проверки состояния `/health`.

Пример:

```
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return { status: 'ok' };
  }
}
```

---

### Инструменты:

-  **PM2** или **Kubernetes** для zero-downtime.

-  **GitHub Actions** или **GitLab CI/CD** для автоматизации.

-  **NGINX/Traefik** для управления маршрутизацией трафика.

Используй этот процесс для надёжного деплоя!