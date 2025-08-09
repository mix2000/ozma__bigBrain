---
order: 1
title: pm2
---

Запустить проект Next.js через `pm2` с определённой версией `nodejs`, установленной через `nvm`, можно следующим образом:

### 1\. Убедитесь, что нужная версия Node.js установлена через `nvm`

```
nvm install 16  # Замените 16 на вашу нужную версию
nvm use 16
```

### 2\. Установите `pm2`, если он ещё не установлен

```
npm install -g pm2
```

### 3\. Создайте файл `ecosystem.config.js`

Этот файл нужен для управления процессами через `pm2`, указав нужную версию Node.js:

```
module.exports = {
  apps: [
    {
      name: "my-next-app",
      script: "node_modules/next/dist/bin/next",
      args: "start",
      env: {
        NODE_ENV: "production",
      },
      interpreter: "~/.nvm/versions/node/v16.20.0/bin/node", // Укажите вашу версию Node.js
      cwd: "/path/to/your/nextjs/project",
    },
  ],
};
```

**Важно:** путь к `node` можно уточнить командой:

```
nvm which 16
```

(или замените `16` на нужную версию).

### 4\. Запустите приложение через pm2

```
pm2 start ecosystem.config.js
```

### 5\. Сохраните процессы и настройте автозапуск

```
pm2 save
pm2 startup
```

### 6\. Управление процессом

-  Перезапуск:

   ```
   pm2 restart my-next-app
   ```

-  Логирование:

   ```
   pm2 logs my-next-app
   ```

-  Остановка:

   ```
   pm2 stop my-next-app
   ```

-  Удаление:

   ```
   pm2 delete my-next-app
   ```


