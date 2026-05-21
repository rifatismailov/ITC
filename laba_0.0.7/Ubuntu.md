# Практичне заняття №33–34
## Тема: Захист Ubuntu-сервера — SSH · UFW · fail2ban

**Дисципліна:** Захист ІТС · КБ бак21  
**Дата:** 22.05.2026  
**Тривалість:** 90 + 90 хвилин  
**Оцінка:** 8 балів

---

## Що налаштовуємо

| Завдання | Що робимо | Результат |
|----------|-----------|-----------|
| А | SSH: зміна порту, вимкнення root | Порт 2222, root заборонено |
| Б | SSH: ключова автентифікація | Вхід без пароля |
| В | UFW: firewall default-deny | Відкриті тільки потрібні порти |
| Г | fail2ban: захист від brute-force | Автоблокування після 3 спроб |

---

## Завдання А — SSH: зміна порту та вимкнення root ⏱ 30 хв

### 1. Підключіться до свого Ubuntu

```bash
ssh student@<your-ip>
```

### 2. Перевірте що є зараз

```bash
sudo sshd -T | grep -E '^port|^permitrootlogin|^passwordauthentication'
```

📝 Запишіть результат:
```
port =
permitrootlogin =
passwordauthentication =
```

### 3. Відредагуйте конфіг SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Знайдіть і змініть ці рядки (якщо рядок закоментований `#` — приберіть `#`):

```
Port 2222
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers student
```

Зберегти: **Ctrl+O → Enter → Ctrl+X**

### 4. Перезапустіть SSH

> ⚠️ Не закривайте поточний термінал! Відкрийте новий для тесту.

```bash
sudo systemctl restart ssh
sudo systemctl status ssh
```

### 5. Відкрийте новий термінал і підключіться через порт 2222

```bash
ssh -p 2222 student@<your-ip>
```

📝 Підключення успішне? (Так / Ні): ___________

### 6. Перевірте що зміни застосувались

```bash
sudo sshd -T | grep -E '^port|^permitrootlogin'
```

📝 Новий результат:
```
port =
permitrootlogin =
```

---

## Завдання Б — SSH: ключова автентифікація ed25519 ⏱ 30 хв

### 1. Згенеруйте ключ на вашому ПК

> ⚠️ Виконувати на вашому комп'ютері, НЕ на сервері!

```bash
ssh-keygen -t ed25519 -C "student_lab"
# На всі питання натискайте Enter
```

Перевірте що ключ створений:
```bash
ls ~/.ssh/
# Має бути: id_ed25519  та  id_ed25519.pub
```

### 2. Скопіюйте публічний ключ на сервер

```bash
ssh-copy-id -p 2222 student@<your-ip>
# Введіть пароль останній раз
```

### 3. Протестуйте вхід без пароля

```bash
ssh -p 2222 student@<your-ip>
# Повинен підключитися БЕЗ запиту пароля
```

📝 Вхід без пароля працює? (Так / Ні): ___________

### 4. Вимкніть паролі (тільки після успішного тесту!)

На сервері:
```bash
sudo nano /etc/ssh/sshd_config
```

Знайдіть і змініть:
```
PasswordAuthentication no
```

```bash
sudo systemctl restart ssh
```

### 5. Перевірте що пароль більше не працює

```bash
# Спроба підключитися з іншого ПК або явно з паролем:
ssh -p 2222 -o PreferredAuthentications=password student@<your-ip>
# Повинно відмовити: Permission denied (publickey)
```

📝 Відповідь сервера: ___________

---

## Завдання В — UFW: firewall ⏱ 25 хв

### 1. Встановіть правила default-deny

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 2. Дозвольте тільки потрібні порти

```bash
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp   comment 'HTTP'
sudo ufw allow 443/tcp  comment 'HTTPS'
sudo ufw deny  3306/tcp comment 'MySQL'
```

### 3. Увімкніть та перевірте

```bash
sudo ufw enable
sudo ufw status verbose
```

📝 Вставте вивід команди:
```
(результат ufw status verbose)
```

### 4. Перевірте стани портів

```bash
# Подивитись що слухає зсередини
sudo ss -tlnp

# Перевірити конкретний порт
sudo ufw status | grep 3306
```

📝 Заповніть таблицю:

| Порт | Правило UFW | Очікуваний стан при скануванні |
|------|-------------|-------------------------------|
| 2222/tcp | allow | open |
| 80/tcp | allow | open |
| 3306/tcp | deny | filtered |
| 8080/tcp | немає | closed |

### 5. Перевірте логування UFW

```bash
sudo ufw logging on
sudo tail -f /var/log/ufw.log
# Спробуйте підключитися на заблокований порт з іншого терміналу
# Ctrl+C щоб зупинити
```

📝 Що з'явилося в лозі при спробі підключитися на порт 3306?
```
(вставте рядок з лога)
```

---

## Завдання Г — fail2ban: захист від brute-force ⏱ 35 хв

### 1. Встановіть fail2ban

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban
```

📝 Статус: ___________

### 2. Створіть конфіг

```bash
sudo nano /etc/fail2ban/jail.local
```

Вставте:

```ini
[DEFAULT]
bantime  = 300
findtime = 120
maxretry = 3

[sshd]
enabled = true
port    = 2222
logpath = /var/log/auth.log
```

Зберегти: **Ctrl+O → Enter → Ctrl+X**

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

📝 Результат команди status sshd:
```
(вставте вивід)
```

### 3. Спровокуйте бан

> ⚠️ Виконуйте з ІНШОГО ПК або терміналу — інакше заблокуєте себе!

```bash
# Тричі введіть невірний пароль
ssh -p 2222 student@<your-ip>
# Пароль: wrongpassword (3 рази)
```

Перевірте бан:
```bash
sudo fail2ban-client status sshd
```

📝 IP в "IP list" (заблокований): ___________

### 4. Переглянути лог та розблокувати

```bash
# Переглянути що сталось
sudo tail -20 /var/log/fail2ban.log

# Розблокувати свій IP
sudo fail2ban-client set sshd unbanip <заблокований-IP>

# Перевірити що розблоковано
sudo fail2ban-client status sshd
```

📝 Рядки з лога (Ban та Unban):
```
(вставте рядки)
```

---

## ✅ Фінальна перевірка

Виконайте всі команди і запишіть результати:

```bash
echo "=== SSH ===" && sudo sshd -T | grep -E 'port|permitroot|passwordauth'
echo "=== UFW ===" && sudo ufw status
echo "=== fail2ban ===" && sudo fail2ban-client status sshd
```

📝 Заповніть чек-ліст:

| Захід | Команда перевірки | Виконано |
|-------|-------------------|----------|
| SSH порт = 2222 | `sudo sshd -T \| grep ^port` | ✓ / ✗ |
| PermitRootLogin = no | `sudo sshd -T \| grep permitrootlogin` | ✓ / ✗ |
| PasswordAuthentication = no | `sudo sshd -T \| grep passwordauth` | ✓ / ✗ |
| UFW активний | `sudo ufw status \| head -1` | ✓ / ✗ |
| UFW: порт 3306 deny | `sudo ufw status \| grep 3306` | ✓ / ✗ |
| fail2ban: sshd jail активний | `sudo fail2ban-client status sshd` | ✓ / ✗ |
| Вхід по ключу працює | `ssh -p 2222 student@<ip>` | ✓ / ✗ |

---

## Висновок

_Опишіть що налаштували та як кожен захід захищає сервер (3–4 речення):_

___________________________________________________________________________

___________________________________________________________________________

___________________________________________________________________________

___________________________________________________________________________

---

## Оцінювання

| Бали | Критерій |
|------|----------|
| 2 | Завдання А+Б: SSH на 2222, root заборонено, ключ працює |
| 2 | Завдання В: UFW активний, правила налаштовані, лог показує блокування |
| 2 | Завдання Г: fail2ban запущено, бан спровоковано і знято |
| 2 | Чек-ліст заповнений + висновок |
| **8** | **Разом** |

> Мінімум для зарахування: **5 балів**
