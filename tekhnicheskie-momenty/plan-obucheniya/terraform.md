---
order: 1
title: Terraform
---

Terraform -- мощный инструмент для управления инфраструктурой как кодом (IaC). Чтобы разобраться в нем, предлагаю следовать такому порядку обучения:

---

### **1\. Введение в Terraform**

-  Что такое **Terraform** и зачем он нужен?

-  Разница между **IaC** (Infrastructure as Code) и традиционным управлением инфраструктурой.

-  Основные **понятия**:

   -  **Providers** (AWS, GCP, Azure, Kubernetes и др.)

   -  **Resources** (создаваемые объекты)

   -  **State** (состояние инфраструктуры)

   -  **Modules** (модульность конфигурации)

   -  **Variables** (переменные)

   -  **Outputs** (выходные данные)

---

### **2\. Установка и настройка**

-  Установка Terraform: скачивание с [официального сайта](https://developer.hashicorp.com/terraform/downloads)

-  Проверка версии:

   ```
   terraform version
   
   ```

-  Базовая настройка для работы с разными облачными провайдерами (AWS, GCP, Azure).

---

### **3\. Основы работы с Terraform**

-  Создание **простейшего** Terraform-проекта:

   1. Инициализация (`terraform init`)

   2. Планирование (`terraform plan`)

   3. Применение (`terraform apply`)

   4. Удаление (`terraform destroy`)

-  Работа с конфигурационными файлами:

   -  [`main.tf`](http://main.tf) (основная конфигурация)

   -  [`variables.tf`](http://variables.tf) (переменные)

   -  `terraform.tfvars` (значения переменных)

---

### **4\. Переменные и Outputs**

-  Объявление переменных (`variable`)

-  Использование значений по умолчанию

-  Передача переменных через `terraform.tfvars` или `-var`

-  Определение выходных данных (`output`)

---

### **5\. Состояние Terraform (State)**

-  Что такое `terraform.tfstate`?

-  Локальное и удаленное хранение состояния (S3, GCS, Terraform Cloud)

-  Команда `terraform state list` и `terraform state show`

-  Управление дрейфом (drift) и обновлением ресурсов

---

### **6\. Модули**

-  Что такое **модуль** и зачем он нужен?

-  Как создавать и использовать модули

-  Встраивание модулей (`source`, `module`)

-  Примеры использования модулей из **Terraform Registry**

---

### **7\. Работа с разными провайдерами**

-  AWS, GCP, Azure, Kubernetes

-  Как подключать разных провайдеров через `provider` блок

-  Настройка аутентификации (IAM, Service Accounts, kubeconfig)

---

### **8\. Управление зависимостями**

-  `depends_on` и порядок создания ресурсов

-  Жизненный цикл ресурсов (`lifecycle`):

   -  `create_before_destroy`

   -  `prevent_destroy`

   -  `ignore_changes`

---

### **9\. Организация и безопасность**

-  Разделение окружений (dev/staging/prod)

-  Использование `backend` для хранения состояния

-  Шифрование данных (KMS, Vault)

-  Работа с **Terraform Cloud**

---

### **10\. CI/CD для Terraform**

-  Автоматизация через **GitHub Actions, GitLab CI, Jenkins**

-  Интеграция с `terraform fmt`, `terraform validate`, `terraform plan`

-  Автоматическое развертывание через **Terraform Apply**

---

### **11\. Дополнительные темы**

-  Динамические блоки (`dynamic`)

-  Работа с **for_each** и **count**

-  Использование **data sources**

-  Debugging и логирование (`TF_LOG`)

---

### **Как учиться?**

1. **Практиковаться**: создать простые конфигурации, а затем наращивать сложность.

2. **Читать документацию**: [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)

3. **Разбирать примеры**: изучать **Terraform Registry** и GitHub-репозитории.

4. **Проходить курсы**:

   -  HashiCorp Certified: Terraform Associate (для сертификации)

   -  Бесплатные уроки на YouTube, Udemy, LinkedIn Learning.

---

Хочешь начать с практики? Могу помочь создать первый [`main.tf`](http://main.tf) для AWS или другого провайдера. 🚀