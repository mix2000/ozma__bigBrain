---
order: 1
title: Оптимизация стилей react
---

Хорошо, давай доработаем первый вариант с учётом того, что у тебя есть три модуля: `desktop.module.scss`, `common.module.scss` и `mobile.module.scss`. Мы будем динамически загружать стили в зависимости от ширины экрана и использовать общие стили, которые загружаются всегда.

---

### **Шаги для реализации:**

1. **Создадим кастомный хук**, который будет загружать нужные стили в зависимости от ширины экрана.

2. **Загрузим общие стили**, которые будут использоваться везде.

3. **Загрузим мобильные или десктопные стили** в зависимости от типа устройства.

4. **Поддержим динамическую загрузку** стилей при изменении размера экрана.

---

### **Реализация кастомного хука**

```
import { useEffect, useState } from "react";

// Тип для возвращаемых стилей
interface Styles {
  common: { [key: string]: string };
  mobile?: { [key: string]: string };
  desktop?: { [key: string]: string };
}

export function useResponsiveStyles() {
  const [styles, setStyles] = useState<Styles | null>(null);

  // Функция для загрузки стилей в зависимости от устройства
  const loadStyles = async () => {
    const commonStyles = await import("./common.module.scss");
    
    let mobileStyles = {};
    let desktopStyles = {};

    if (window.matchMedia("(max-width: 768px)").matches) {
      mobileStyles = await import("./mobile.module.scss");
    } else {
      desktopStyles = await import("./desktop.module.scss");
    }

    setStyles({
      common: commonStyles.default,
      mobile: mobileStyles?.default || {},
      desktop: desktopStyles?.default || {},
    });
  };

  // Запуск загрузки стилей при монтировании и изменении размера экрана
  useEffect(() => {
    loadStyles();
    const handleResize = () => loadStyles();
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return styles;
}
```

---

### **Использование в компоненте**

Теперь можно использовать этот хук в компоненте и получать стили для разных версий устройства:

```
import { useResponsiveStyles } from "./useResponsiveStyles";

function MyComponent() {
  const styles = useResponsiveStyles();

  if (!styles) return null; // Пока стили загружаются, можно показывать прелоадер

  return (
    <div className={styles.common.container}>
      <div className={styles.mobile?.title || styles.desktop?.title}>
        Контент для мобильных/десктопных версий
      </div>
    </div>
  );
}
```

---

### **Как это работает:**

1. **Общие стили** загружаются всегда -- они есть в `common.module.scss`.

2. **Мобильные стили** загружаются только для мобильных устройств, когда ширина экрана меньше 768px.

3. **Десктопные стили** загружаются для всех остальных устройств.

4. Хук `useResponsiveStyles` следит за изменением ширины экрана с помощью `resize` события, что позволяет динамически переключать стили, когда пользователь меняет размер окна браузера.

---

### **Преимущества такого подхода:**

-  **Оптимизация загрузки**: стили для мобильных и десктопных версий подгружаются только по мере необходимости.

-  **Гибкость**: легко масштабируемо для добавления новых стилей для других устройств или версий.

-  **Отдельное управление стилями** для мобильных, десктопных и общих элементов.

Если нужно добавить дополнительные условия или стили для других версий устройств, просто добавь дополнительные файлы стилей и логику в хук.



В этом случае мы можем передавать путь к стилям в хук, чтобы он мог загрузить правильные стили для каждого компонента. Ты прав, нужно динамически формировать путь в зависимости от структуры проекта.

### **Как это можно сделать:**

1. Мы будем передавать путь к стилям в хук для каждого компонента.

2. Хук будет использовать этот путь для загрузки нужных файлов стилей.

3. Это позволит управлять стилями для каждого компонента отдельно (например, для `Header`, `Button`, и т.д.), что даёт большую гибкость.

### **1\. Модификация хука для поддержки параметров пути**

```
import { useEffect, useState } from "react";

// Тип для возвращаемых стилей
interface Styles {
  common: { [key: string]: string };
  mobile?: { [key: string]: string };
  desktop?: { [key: string]: string };
}

export function useResponsiveStyles(component: string) {
  const [styles, setStyles] = useState<Styles | null>(null);

  // Функция для загрузки стилей в зависимости от устройства
  const loadStyles = async () => {
    const commonStyles = await import(`./${component}/common.module.scss`);
    
    let mobileStyles = {};
    let desktopStyles = {};

    if (window.matchMedia("(max-width: 768px)").matches) {
      mobileStyles = await import(`./${component}/mobile.module.scss`);
    } else {
      desktopStyles = await import(`./${component}/desktop.module.scss`);
    }

    setStyles({
      common: commonStyles.default,
      mobile: mobileStyles?.default || {},
      desktop: desktopStyles?.default || {},
    });
  };

  // Запуск загрузки стилей при монтировании и изменении размера экрана
  useEffect(() => {
    loadStyles();
    const handleResize = () => loadStyles();
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, [component]); // Перезагружать стили, если компонент изменился

  return styles;
}
```

### **2\. Использование хука в компоненте**

Теперь можно использовать хук для разных компонентов, передавая в него путь к нужным стилям для каждого компонента. Например, для компонента `Header` и `Button`:

```
// Компонент Header
import { useResponsiveStyles } from "./useResponsiveStyles";

function Header() {
  const styles = useResponsiveStyles("header");

  if (!styles) return null; // Пока стили загружаются

  return (
    <div className={styles.common.container}>
      <h1 className={styles.mobile?.title || styles.desktop?.title}>
        Привет, мир!
      </h1>
    </div>
  );
}

// Компонент Button
import { useResponsiveStyles } from "./useResponsiveStyles";

function Button() {
  const styles = useResponsiveStyles("button");

  if (!styles) return null; // Пока стили загружаются

  return (
    <button className={styles.common.btn}>
      Нажми меня
    </button>
  );
}
```

### **3\. Структура проекта**

В этом примере структура проекта может быть такой:

```
/components
  /Button
    button.module.scss
    mobile.module.scss
    desktop.module.scss
  /Header
    header.module.scss
    mobile.module.scss
    desktop.module.scss
  /common.module.scss
```

### **Объяснение:**

-  **Путь к стилям** передаётся через параметр `component` в хук `useResponsiveStyles`. Например, для компонента `Button` передается путь `"button"`, для компонента `Header` -- `"header"`.

-  **Общие стили** подгружаются из общего файла `common.module.scss`.

-  Для **мобильных** и **десктопных версий** стили подгружаются из соответствующих файлов в папке компонента.

### **Преимущества подхода:**

1. **Изоляция стилей** для каждого компонента с учётом его контекста (например, мобильные стили для `Header`, десктопные для `Button`).

2. **Гибкость** -- можно легко добавлять новые компоненты с их собственными стилями и подгружать их в зависимости от устройства.

3. **Оптимизация** -- стили загружаются только тогда, когда они действительно нужны, что помогает уменьшить размер бандла.

Такой подход позволяет эффективно управлять стилями, избегая ненужной загрузки и сохраняя хорошую изоляцию каждого компонента.