---
order: 2
title: Переупаковка и публикация пакета в GitHub Package Registry с изменением имени
---

### Переупаковка и публикация пакета в GitHub Package Registry с изменением имени

Тебе нужно сделать следующее:

---

## **1\. Скачай существующий пакет**

Так как у тебя уже установлен пакет `@nord-beaver/eslint-config-base`, можешь скачать его исходники:

```
npm pack @nord-beaver/eslint-config-base
```

Эта команда создаст `.tgz` архив с кодом пакета.

Распакуй его:

```
tar -xzf nord-beaver-eslint-config-base-<version>.tgz
```

Теперь у тебя есть исходный код пакета.

---

## **2\. Измени** `package.json`

Открой распакованный пакет и отредактируй `package.json`:

-  Измени `name` на новый `@tennisigit/eslint-config-base`

-  Убедись, что `publishConfig.registry` указывает на [`https://npm.pkg.github.com/`](https://npm.pkg.github.com/)

Пример:

```
{
  "name": "@tennisigit/eslint-config-base",
  "version": "1.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com/"
  }
}
```

---

## **3\. Авторизуйся в GitHub Package Registry**

Убедись, что у тебя есть токен GitHub с правами `write:packages` и `read:packages`. Затем настрой `.npmrc`:

```
npm login --registry=https://npm.pkg.github.com
```

Введи свои данные:

-  `Username`: твой GitHub username

-  `Password`: твой GitHub-токен (не GitHub-пароль)

-  `Email`: твоя почта в GitHub

---

## **4\. Запакуй и опубликуй новый пакет**

Теперь запусти команду упаковки и публикации:

```
npm pack
npm publish --registry=https://npm.pkg.github.com
```

Если всё прошло успешно, ты увидишь сообщение о публикации.

---

## **5\. Установка пакета**

Теперь ты можешь установить свой пакет:

```
npm install @tennisigit/eslint-config-base --registry=https://npm.pkg.github.com
```

Или добавить в `.npmrc` настройку, чтобы не указывать каждый раз `--registry`:

```
@tennisigit:registry=https://npm.pkg.github.com/
```

После этого можно устанавливать пакет как обычно:

```
npm install @tennisigit/eslint-config-base
```



Похоже, что у тебя в `package.json` в секции `scripts` есть команда `publish`, которая снова вызывает `npm publish`, из-за чего публикация выполняется дважды.

### **Как исправить?**

1. **Проверь** `package.json`\
   Открой `package.json` и найди секцию `scripts`:

   ```
   "scripts": {
     "publish": "npm publish"
   }
   
   ```

   Если там есть эта строка, **удали её** или замени на что-то более полезное, например:

   ```
   "scripts": {
     "prepublishOnly": "echo 'Preparing for publishing...'"
   }
   
   ```

2. **Попробуй запустить команду без скрипта**\
   Если ты запускаешь `npm publish` напрямую и всё ещё видишь двойное выполнение, попробуй выполнить его с флагом `--ignore-scripts`:

   ```
   npm publish --ignore-scripts
   
   ```

   Это предотвратит запуск `prepublish`, `prepare`, `prepublishOnly` или других связанных команд.

3. **Очисть кэш npm**\
   Если проблема сохраняется, попробуй очистить кэш npm:

   ```
   npm cache clean --force
   
   ```

4. **Обнови версию пакета**\
   Ошибка `EPUBLISHCONFLICT` говорит о том, что эта версия уже загружена. Обнови `package.json`, увеличив номер версии, например:

   ```
   npm version patch
   
   ```

   Затем попробуй снова:

   ```
   npm publish
   
   ```

---

После этих шагов `npm publish` больше не должен запускаться дважды. Попробуй и сообщи, если проблема останется! 🚀