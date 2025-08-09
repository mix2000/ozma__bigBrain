---
order: 1
title: supervisor
---

Если твое приложение NestJS падает и не перезапускается при запуске через `supervisorctl`, тебе нужно проверить несколько вещей:

---

## 1\. **Проверка логов Supervisor**

Supervisor обычно записывает логи процессов, которые он управляет. Найди конфигурацию Supervisor для твоего приложения (обычно в `/etc/supervisor/conf.d/myapp.conf`) и посмотри, где указываются логи:

```
[program:myapp]
command=node dist/main.js
autostart=true
autorestart=true
stderr_logfile=/var/log/myapp_err.log
stdout_logfile=/var/log/myapp_out.log
```

### Проверь ошибки в логах:

```
cat /var/log/myapp_err.log
cat /var/log/myapp_out.log
```

Или можешь смотреть логи в реальном времени:

```
tail -f /var/log/myapp_err.log
```

---

## 2\. **Проверка логов NestJS**

Если Supervisor не даёт тебе достаточно информации, добавь логирование в приложение NestJS.

### Добавь логирование в `main.ts`

```
import { Logger } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const port = process.env.PORT || 3000;
  await app.listen(port);

  Logger.log(`Application is running on: http://localhost:${port}`);
}

bootstrap().catch((err) => {
  Logger.error('Error during app bootstrap:', err);
});
```

Запусти приложение вручную:

```
node dist/main.js
```

Если ошибка есть, ты её увидишь сразу.

---

## 3\. **Проверка статуса в Supervisor**

Выполни команду:

```
supervisorctl status
```

Если приложение остановлено, попробуй его перезапустить вручную:

```
supervisorctl restart myapp
```

Если оно сразу же падает, посмотри статус снова и проверь, нет ли там причин остановки.

---

## 4\. **Посмотри глобальные логи Supervisor**

Если не получается понять, почему падает, проверь основной лог Supervisor:

```
cat /var/log/supervisor/supervisord.log
```

---

## 5\. **Включи autorestart и limit**

Если твое приложение падает из-за ошибки, но Supervisor не перезапускает его, проверь конфиг:

```
[program:myapp]
command=node dist/main.js
autostart=true
autorestart=true
startretries=5
stderr_logfile=/var/log/myapp_err.log
stdout_logfile=/var/log/myapp_out.log
```

После изменений не забудь перечитать конфиги и перезапустить Supervisor:

```
supervisorctl reread
supervisorctl update
supervisorctl restart myapp
```

---

## 6\. **Отлов неожиданных ошибок в NestJS**

Добавь глобальный обработчик ошибок:

```
import { ArgumentsHost, Catch, ExceptionFilter, Logger } from '@nestjs/common';

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(GlobalExceptionFilter.name);

  catch(exception: any, host: ArgumentsHost) {
    this.logger.error('Unhandled exception', exception);
  }
}
```

И подключи его в `main.ts`:

```
app.useGlobalFilters(new GlobalExceptionFilter());
```

---

## 7\. **Проверь использование памяти**

Если приложение падает из-за нехватки памяти, посмотри логи:

```
dmesg | grep -i 'oom'
```

Если есть `Out of memory`, попробуй увеличить память или использовать PM2 вместо Supervisor.

---

### Итог

1. Проверь логи Supervisor (`/var/log/myapp_err.log`).

2. Запусти приложение вручную (`node dist/main.js`).

3. Посмотри `supervisorctl status`.

4. Включи `autorestart=true` в конфиге Supervisor.

5. Добавь глобальный обработчик ошибок в NestJS.



Supervisor перезапускает твое приложение `msExcel`, потому что оно получает сигнал `SIGQUIT`, что указывает на **принудительное завершение процесса**. Возможные причины:

## 1\. **Приложение само завершает работу**

NestJS может завершаться из-за:

-  Некорректных обработчиков ошибок.

-  Отсутствия обработки `unhandledRejection` или `uncaughtException`.

-  Условий выхода в коде (например, `process.exit(1)`).

🔹 **Решение:** Добавь глобальную обработку ошибок в `main.ts`:

```
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
});
```

---

## 2\. **Ограничение по памяти (OOM)**

Система может убивать процесс из-за нехватки памяти.

🔹 **Проверь логи ядра:**

```
dmesg | grep -i 'oom'
```

Если там есть строки типа `Out of memory`, то причина в нехватке RAM.

🔹 **Решение:**

-  Увеличить объем памяти.

-  Использовать `PM2` вместо `Supervisor`, так как у него есть защита от утечек памяти.

---

## 3\. **Supervisor настроен на остановку по ошибкам**

Если в конфигурации `supervisor.conf` указаны условия `exitcodes`, `stopwaitsecs`, Supervisor может останавливать процесс.

🔹 **Проверь конфиг:**

```
cat /etc/supervisor/conf.d/msExcel.conf
```

Если там есть:

```
exitcodes=0
stopwaitsecs=10
```

Попробуй убрать `exitcodes` или увеличить `stopwaitsecs`.

---

## 4\. **Ошибка в коде**

Если приложение падает из-за ошибки в коде, попробуй запустить его вручную:

```
node dist/main.js
```

Если есть ошибки, они будут видны.

---

## 5\. **Проблемы с зависимостями**

Если приложение использует `ExcelJS` или другие модули, они могут падать из-за зависимостей.

🔹 **Решение:** Попробуй переустановить зависимости:

```
rm -rf node_modules package-lock.json
npm install
```

---

## Вывод

Наиболее вероятная причина -- либо **система завершает процесс (**`OOM`**)**, либо **NestJS падает из-за ошибки**. Проверь:

1. **Логи Supervisor** (`/var/log/msExcel_err.log`).

2. **Логи ядра на OOM** (`dmesg | grep -i 'oom'`).

3. **Запуск вручную** (`node dist/main.js`).

4. **Конфиг Supervisor** (`cat /etc/supervisor/conf.d/msExcel.conf`).

Дай мне больше логов, если причина не найдена. 🚀