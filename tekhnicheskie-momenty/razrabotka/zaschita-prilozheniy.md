---
order: 3
title: Защита приложений
---

Защита чувствительных данных -- это комбинация технических, организационных и инфраструктурных мер. Разделю их на несколько уровней:

---

### **1\. Безопасность хранения данных**

#### **1\.1 Шифрование базы данных**

-  **At-Rest (в состоянии покоя):**

   -  **PostgreSQL** --> Transparent Data Encryption (TDE) или pgcrypto.

   -  **MySQL** --> InnoDB TDE.

   -  **MongoDB** --> Encrypted Storage Engine.

   -  **AWS RDS / GCP Cloud SQL** --> Встроенное шифрование.

-  **In-Transit (при передаче)**

   -  Использовать **TLS 1.2+** для передачи данных.

   -  Включить **SSL-соединение** между сервисами.

#### **1\.2 Шифрование на уровне полей**

-  Если нужно шифровать отдельные поля, можно использовать **AES-256**:

   ```
   import crypto from 'crypto';
   
   const algorithm = 'aes-256-gcm';
   const key = crypto.randomBytes(32);
   const iv = crypto.randomBytes(16);
   
   function encrypt(text: string) {
       const cipher = crypto.createCipheriv(algorithm, key, iv);
       let encrypted = cipher.update(text, 'utf8', 'hex');
       encrypted += cipher.final('hex');
       return encrypted;
   }
   
   ```

-  Для работы с данными без расшифровки можно использовать **Homomorphic Encryption** (но это медленно).

#### **1\.3 Токенизация**

-  Чувствительные данные заменяются токенами (например, **Vault HashiCorp** или **AWS KMS**).

---

### **2\. Контроль доступа**

#### **2\.1 Роли и права (RBAC / ABAC)**

-  **RBAC (Role-Based Access Control)**

   -  Определить роли (`admin`, `manager`, `finance`).

   -  Использовать **Keycloak**, **Casbin** или **NestJS Guards** для управления доступом.

-  **ABAC (Attribute-Based Access Control)**

   -  Ограничения на основе условий (например, IP-адресов, времени, локации).

```
import { CanActivate, ExecutionContext } from '@nestjs/common';

export class RoleGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.user?.role === 'admin';
  }
}
```

#### **2\.2 Ограничение доступа к БД**

-  Использовать **IAM-роли** для подключения к БД вместо паролей.

-  Ограничить доступ **только по внутренним VPN/Private Network**.

-  Включить **audit logging** (кто что запрашивал).

#### **2\.3 MFA (многофакторная аутентификация)**

-  Подключить **Google Authenticator, Authy** или **WebAuthn**.

---

### **3\. Безопасность API**

#### **3\.1 Ограничение доступа**

-  **API Gateway (Kong, Nginx, Apigee)** для rate limiting и аутентификации.

-  **JWT (с коротким TTL) + Refresh Tokens**.

-  **OAuth2.0 / OpenID Connect** -- если работаешь с внешними сервисами.

#### **3\.2 Подпись запросов**

-  Использовать **HMAC (SHA-256)** или **JWT** с секретным ключом.

```
import crypto from 'crypto';

function signRequest(data: string, secret: string) {
    return crypto.createHmac('sha256', secret).update(data).digest('hex');
}
```

---

### **4\. Логирование и мониторинг**

#### **4\.1 Защита логов**

-  Никогда не логировать **пароли, токены, номера карт**.

-  Включить **обфускацию логов**.

-  Использовать **SIEM-системы (Splunk, Graylog, ELK Stack)**.

#### **4\.2 Контроль аномалий**

-  **Prometheus + Grafana** -- для мониторинга подозрительной активности.

-  **Fail2Ban** -- для блокировки IP после множества неудачных попыток входа.

---

### **5\. Физическая и организационная защита**

-  **Хранение секретов**

   -  Использовать **Vault HashiCorp** или **AWS Secrets Manager** для ключей API.

-  **Защита серверов**

   -  Доступ к серверам только через **SSH с ключами**, без паролей.

   -  Включить **audit trail** для всех действий.

---

### **6\. План реагирования на инциденты**

-  **Настроить алерты** на попытки несанкционированного доступа.

-  **Периодически проверять утечки данных** (использовать `haveibeenpwned` API).

-  **Регулярные бэкапы** в **защищённом хранилище** (с шифрованием).

---

Если бы у тебя была конкретная угроза (например, инсайдеры, атаки снаружи, утечки), можно было бы рассмотреть конкретные меры. Сейчас это базовый набор, который можно масштабировать в зависимости от критичности. Какой аспект тебе критичнее всего?