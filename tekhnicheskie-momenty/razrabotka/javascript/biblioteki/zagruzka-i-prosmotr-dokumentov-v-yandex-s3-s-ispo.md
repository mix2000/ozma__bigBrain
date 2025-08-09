---
order: 2
title: >-
  Загрузка и просмотр документов в Yandex S3 с использованием Node.js и
  TypeScript
---

Для работы с **Yandex Object Storage (S3)** в **Node.js** удобно использовать библиотеку `aws-sdk`, так как Yandex S3 совместим с API Amazon S3.

#### 1\. Установка зависимостей

```
npm install aws-sdk
```

#### 2\. Конфигурация клиента Yandex S3

Создай файл `.env` и добавь:

```
YANDEX_ACCESS_KEY=your-access-key
YANDEX_SECRET_KEY=your-secret-key
YANDEX_BUCKET_NAME=your-bucket-name
YANDEX_ENDPOINT=https://storage.yandexcloud.net
```

#### 3\. Функция загрузки файла в Yandex S3

```
import { S3 } from 'aws-sdk';
import fs from 'fs';
import path from 'path';
import dotenv from 'dotenv';

dotenv.config();

const s3 = new S3({
  endpoint: process.env.YANDEX_ENDPOINT,
  accessKeyId: process.env.YANDEX_ACCESS_KEY,
  secretAccessKey: process.env.YANDEX_SECRET_KEY,
  region: 'ru-central1',
  s3ForcePathStyle: true,
});

async function uploadFile(filePath: string, fileKey: string): Promise<string> {
  const fileContent = fs.readFileSync(filePath);
  const fileExtension = path.extname(filePath);

  const params = {
    Bucket: process.env.YANDEX_BUCKET_NAME!,
    Key: fileKey,
    Body: fileContent,
    ContentType: getContentType(fileExtension),
    ACL: 'public-read', // Если нужно, чтобы файл был доступен публично
  };

  await s3.upload(params).promise();

  return `${process.env.YANDEX_ENDPOINT}/${process.env.YANDEX_BUCKET_NAME}/${fileKey}`;
}

function getContentType(ext: string): string {
  const mimeTypes: Record<string, string> = {
    '.pdf': 'application/pdf',
    '.doc': 'application/msword',
    '.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    '.png': 'image/png',
    '.jpg': 'image/jpeg',
  };

  return mimeTypes[ext] || 'application/octet-stream';
}

// Пример использования:
uploadFile('example.pdf', 'documents/example.pdf')
  .then((url) => console.log(`Файл загружен: ${url}`))
  .catch((err) => console.error('Ошибка загрузки:', err));
```

---

#### 4\. Получение ссылки для скачивания/просмотра файла

```
async function getFileUrl(fileKey: string): Promise<string> {
  const params = {
    Bucket: process.env.YANDEX_BUCKET_NAME!,
    Key: fileKey,
    Expires: 60 * 60, // Время действия ссылки (в секундах)
  };

  return s3.getSignedUrlPromise('getObject', params);
}

// Пример использования:
getFileUrl('documents/example.pdf')
  .then((url) => console.log(`Ссылка на файл: ${url}`))
  .catch((err) => console.error('Ошибка получения ссылки:', err));
```

---

### Итог

-  **Загрузка файла:** Используем `uploadFile(filePath, fileKey)`.

-  **Просмотр/скачивание файла:** Если он `public-read`, можно открыть прямую ссылку. Если приватный -- генерируем временную ссылку `getFileUrl(fileKey)`.