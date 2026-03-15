# 🔐 Лабораторна робота №2

## Дискреційний контроль доступу (DAC) у Linux

---

# 📚 Мета роботи

Навчитись керувати доступом до файлів та директорій у Linux за допомогою:

* моделі **DAC (Discretionary Access Control)**
* стандартних прав **Owner / Group / Others**
* розширених **POSIX ACL**
* спеціальних бітів файлової системи

Також дослідити принципи:

* **Least Privilege** — мінімальні привілеї
* **Separation of Duties** — розподіл обов'язків

---

# 🖥 Підготовка середовища

Лабораторна виконується на **будь-якій Linux системі**.

Потрібно встановити пакет для роботи з ACL.

```bash
sudo apt update
sudo apt install acl
```

### Що це за пакет

| Пакет | Призначення                                 |
| ----- | ------------------------------------------- |
| acl   | підтримка розширених прав доступу POSIX ACL |

---

# 👥 Завдання 1 — Створення користувачів та груп

У моделі **DAC** адміністратор створює користувачів та групи, а власники файлів можуть самостійно керувати доступом.

---

## Створення груп

```bash
sudo groupadd developers
sudo groupadd managers
```

---

## Створення користувачів

```bash
# Alice — розробник
sudo useradd -m -s /bin/bash alice
sudo usermod -aG developers alice

# Bob — менеджер
sudo useradd -m -s /bin/bash bob
sudo usermod -aG managers bob

# Charlie — стажер
sudo useradd -m -s /bin/bash charlie
```

---

### 📖 Пояснення

| Команда     | Опис                       |
| ----------- | -------------------------- |
| useradd     | створює користувача        |
| -m          | створює домашню директорію |
| -s          | задає shell                |
| usermod -aG | додає користувача до групи |

---

# 📁 Завдання 2 — Базовий DAC (Owner / Group / Others)

Сценарій:

Розробники мають **спільну папку для коду**, а менеджери не повинні мати доступ до неї.

---

## Створення директорії

```bash
sudo mkdir -p /projects/backend
```

---

## Зміна власника та групи

```bash
sudo chown alice:developers /projects/backend
```

---

## Встановлення прав доступу

```bash
sudo chmod 750 /projects/backend
```

---

### Розшифровка прав

| Користувач         | Права |
| ------------------ | ----- |
| Owner (alice)      | rwx   |
| Group (developers) | r-x   |
| Others             | ---   |

---

## Перевірка доступу

### Доступ для Alice

```bash
su - alice -c "ls /projects/backend"
```

Результат:

```text
Success
```

---

### Доступ для Bob

```bash
su - bob -c "ls /projects/backend"
```

Результат:

```text
Permission denied
```

---

# 🧩 Завдання 3 — Розширений контроль доступу (POSIX ACL)

Іноді стандартних прав **rwx** недостатньо.

Сценарій:

Менеджеру **Bob** потрібно надати **доступ тільки на читання** до одного файлу.

---

## Створення файлу

```bash
su - alice -c "echo 'Secret Code' > /projects/backend/api.py"
```

---

## Перегляд ACL

```bash
getfacl /projects/backend/api.py
```

---

## Додавання доступу для Bob

```bash
sudo setfacl -m u:bob:r /projects/backend/api.py
```

---

## Перевірка

Bob може прочитати файл:

```bash
su - bob -c "cat /projects/backend/api.py"
```

Bob **не може**:

```bash
su - bob -c "nano /projects/backend/api.py"
su - bob -c "touch /projects/backend/newfile.py"
```

---

# 📌 Завдання 4 — Sticky Bit

Sticky Bit використовується для спільних директорій.

---

## Створення спільної папки

```bash
sudo mkdir /projects/tmp_share
sudo chmod 777 /projects/tmp_share
```

---

## Встановлення Sticky Bit

```bash
sudo chmod +t /projects/tmp_share
```

---

## Як це працює

У такій директорії:

* файли можуть створювати **всі користувачі**
* видаляти файл може **лише його власник**

---

## Практика

### Створення файлу

```bash
su - alice
touch /projects/tmp_share/test.txt
```

---

### Спроба видалити файл

```bash
su - bob
rm /projects/tmp_share/test.txt
```

Результат:

```text
Operation not permitted
```

---

# 📖 Теорія

## Discretionary Access Control (DAC)

DAC — модель контролю доступу, у якій **власник ресурсу сам визначає**, хто має доступ до файлу або директорії.

Це стандартна модель безпеки у Linux.

---

## Least Privilege

Користувач повинен мати **мінімально необхідні права**.

Наприклад:

```bash
chmod 600 passwords.txt
```

---

### Значення прав

| Owner | Group | Others |
| ----- | ----- | ------ |
| rw    | ---   | ---    |

---

## Separation of Duties

Безпека підвищується, коли ролі розділені.

| Користувач | Роль      |
| ---------- | --------- |
| Alice      | Developer |
| Bob        | Manager   |
| Charlie    | Intern    |

Якщо один обліковий запис буде зламаний — система залишиться частково захищеною.

---

# 🧪 Контрольні питання

1️⃣ Що таке **DAC**?
Чому ця модель називається *дискреційною*?

---

2️⃣ Принцип **Least Privilege**

Які права доступу найбільш безпечні для файлу з паролями?

---

3️⃣ **ACL vs chmod**

У яких випадках стандартних прав `rwx` недостатньо?

---

4️⃣ **Separation of Duties**

Як розділення користувачів `alice` та `bob` допомагає безпеці системи?

---

# ✅ Результат лабораторної

Було досліджено:

* модель **DAC**
* права доступу **Owner / Group / Others**
* розширені **POSIX ACL**
* механізм **Sticky Bit**

---

# 🎉 Висновок

Було досліджено механізми контролю доступу Linux, які дозволяють гнучко керувати доступом до файлів та директорій.
Ці механізми є фундаментальною частиною **безпеки операційних систем Linux/Unix**.
