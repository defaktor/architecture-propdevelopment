Внутри кластера Kubernetes для **PropDevelopment** требуется развернуть 4 сервиса и разграничить сетевой трафик:

- `front-end`: Витрина сервисов для собственников.
- `back-end-api`: Сервис для бизнес-логики tenant-core-app.
- `admin-front-end`: Интерфейс администратора.
- `admin-back-end-api`: API управления для администраторов (особо чувствительный сервис, изолированный от других).

---

### **1. Создание отдельного namespace**
```bash
kubectl create namespace propdevelopment
```

---

### **2. Создание манифеста для развёртывания сервисов (Deployments)**
`propdev-deployments.yaml`:

Применяем деплойменты:
```bash
kubectl apply -f propdev-deployments.yaml
```

---

### **3. Создание сервисов для доступа к Pod'ам (Service)**
`propdev-services.yaml`:

Создаём сервисы:

```bash
kubectl apply -f propdev-services.yaml
```

---

### **4. Настройка политики сетевой изоляции (NetworkPolicy)**

Создайте файл `network-policy.yaml`.  
Эта политика изолирует доступ к сервису `admin-back-end-api` — никто, кроме `admin-front-end`, не сможет с ним взаимодействовать.
- Другие поды (front-end, back-end-api) не имеют доступа к admin-back-end-api.

Примените политику:

```bash
kubectl apply -f network-policy.yaml
```

---

### **5. Проверка корректности настройки политик**

- Проверьте доступность с помощью временных Pod’ов:

Проверка доступа от `admin-front-end` (должен быть доступ):

```bash
kubectl -n propdevelopment exec -it $(kubectl -n propdevelopment get pods -l app=admin-front-end -o jsonpath='{.items[0].metadata.name}') -- curl admin-back-end-api-svc
```

- Должен быть ответ от сервиса (например, приветственная страница nginx).
Проверка от другого сервиса (должна быть заблокирована):

```bash
kubectl -n propdevelopment exec -it $(kubectl -n propdevelopment get pods -l app=front-end -o jsonpath='{.items[0].metadata.name}') -- curl --connect-timeout 5 admin-back-end-api-svc
```

- Запрос должен завершиться ошибкой (таймаут).


### **Создание сетевых политик**
`non-admin-api-allow.yaml`

```bash
kubectl apply -f non-admin-api-allow.yaml
```

---

### **Проверка сетевых политик:**

#### Проверка доступности между сервисами, которым доступ разрешён:

Например, проверка доступа от `front-end` к `back-end-api`:

```bash
kubectl -n propdevelopment exec -it $(kubectl -n propdevelopment get pods -l app=front-end -o jsonpath='{.items[0].metadata.name}') -- curl --connect-timeout 5 back-end-api-svc
```

_Запрос должен завершиться успешно._

Проверка от `admin-front-end` к `admin-back-end-api`:

```bash
kubectl -n propdevelopment exec -it $(kubectl -n propdevelopment get pods -l app=admin-front-end -o jsonpath='{.items[0].metadata.name}') -- curl --connect-timeout 5 admin-back-end-api-svc
```

_Запрос должен завершиться успешно._

---

### 🛡️ **Проверка изоляции**

Например, доступ от `front-end` к `admin-back-end-api`:

```bash
kubectl -n propdevelopment exec -it $(kubectl -n propdevelopment get pods -l app=front-end -o jsonpath='{.items[0].metadata.name}') -- curl --connect-timeout 5 admin-back-end-api-svc
```

_Запрос должен завершиться ошибкой (таймаут)._
