# 🔧 Лабораторна робота: Віддалений доступ, Docker та Portainer за OPNsense

## Топологія мережі

```
[Зовнішня мережа] → [OPNsense WAN] → [OPNsense LAN 192.168.1.1] → [Ubuntu Server 192.168.1.XXX]
```

| Пристрій | Роль | IP-адреса |
|---|---|---|
| OPNsense | Шлюз / Фаєрвол | LAN: `192.168.1.1` |
| Ubuntu Server | Сервер додатків | `192.168.1.XXX` |

> ⚠️ **Зверніть увагу:** IP-адреса Ubuntu Server може відрізнятися у вашому середовищі. Уточніть її перед початком роботи.

![Топологія мережі](<image/Знімок екрана 2026-03-08 о 22.47.51.png>)

---

## Крок 1. Налаштування Port Forwarding для SSH на OPNsense

### 1.1 Створення правила NAT Port Forward

1. Перейдіть у **Firewall → NAT → Destination NAT (Port Forward)**
2. Натисніть кнопку **+ Add**
3. Заповніть параметри:

| Параметр | Значення |
|---|---|
| Interface | `WAN` |
| Protocol | `TCP` |
| Destination | `WAN address` |
| Destination port range | `2222` |
| Redirect target IP | `192.168.1.XXX` |
| Redirect target port | `22` |
| Description | `SSH to Ubuntu` |

> ⚠️ **Важливо:** У полі **Redirect Target IP** вкажіть IP-адресу **вашого** Ubuntu Server, а не ту, що зображена на скріншоті нижче.

![Налаштування NAT Port Forward](<image/Знімок екрана 2026-03-08 о 23.13.55.png>)

4. Натисніть **Save**, потім **Apply Changes**

> ℹ️ OPNsense автоматично створить відповідне дозвільне правило у **Firewall → Rules → WAN**. Перевірте його наявність після збереження.

---

## Крок 2. Перевірка та запуск SSH на Ubuntu

Підключіться до Ubuntu через консоль і виконайте перевірку статусу SSH:

```bash
# Перевірити статус SSH
sudo systemctl status ssh
```

**Якщо SSH активний** — одразу переходьте до Кроку 3.

**Якщо SSH не встановлений** — виконайте встановлення:

```bash
# Встановити SSH
sudo apt update && sudo apt install openssh-server -y
```

> ⚠️ **Можлива помилка під час встановлення:** Якщо під час виконання команди виникне помилка (як на зображенні нижче), перезавантажте Ubuntu командою `reboot` та спробуйте встановлення повторно.

![Помилка встановлення SSH](<image/error_ssh.png>)

**Якщо SSH встановлений, але не активний** — запустіть та додайте до автозапуску:

```bash
# Запустити сервіс
sudo systemctl start ssh

# Додати до автозапуску
sudo systemctl enable ssh

# Перевірити що SSH слухає на порту 22
ss -tlnp | grep 22
```

![Перевірка стану SSH](<image/Знімок екрана 2026-03-08 о 22.40.39.png>)

---

## Крок 3. Підключення через SSH з зовнішньої мережі

```bash
# Формат команди
ssh -p <зовнішній_порт> <користувач>@<WAN_IP_OPNsense>

# Приклад
ssh -p 2222 student@192.168.88.XXX
```

> ℹ️ `192.168.88.XXX` — це IP-адреса **WAN-порту вашого OPNsense**, за якою ви підключаєтесь до веб-інтерфейсу OPNsense.

> 💡 Використання нестандартного порту `2222` замість стандартного `22` зменшує кількість автоматизованих атак на SSH.

![Підключення через SSH](<image/Знімок екрана 2026-03-08 о 22.42.32.png>)

---

## Крок 4. Встановлення Docker та Portainer

### 4.1 Встановлення Docker

```bash
# Додаємо офіційний GPG-ключ Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Додаємо офіційний репозиторій Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  bionic stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Оновлюємо список пакетів та встановлюємо Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

![Встановлення Docker](<image/isntallDocker.png>)

### 4.2 Запуск Portainer Community Edition

```bash
# Створити persistent volume для збереження даних Portainer
docker volume create portainer_data

# Запустити контейнер Portainer
sudo docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Перевірити що контейнер запущено
docker ps
```

> 🌐 Portainer буде доступний локально за адресою: `https://192.168.1.XXX:9443`, де `192.168.1.XXX` — IP-адреса вашого Ubuntu Server.

---

## Крок 5. Налаштування UFW на Ubuntu

> ⚠️ **Обов'язково** налаштуйте правила для SSH **до** увімкнення UFW, інакше втратите доступ до сервера!

```bash
# 1. Дозволити SSH тільки з IP шлюзу OPNsense
sudo ufw allow from 192.168.1.1 to any port 22 proto tcp

# 2. Дозволити доступ до Portainer тільки з IP шлюзу OPNsense
sudo ufw allow from 192.168.1.1 to any port 9443 proto tcp

# 3. Увімкнути UFW
sudo ufw enable

# 4. Перевірити правила
sudo ufw status verbose
```

Очікуваний вивід:

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    192.168.1.1
9443/tcp                   ALLOW IN    192.168.1.1
```

![Правило UFW для SSH (порт 22)](<image/Allow_22.png>)

![Правило UFW для Portainer (порт 9443)](<image/Allow_9443.png>)

---

## Крок 6. Port Forwarding для Portainer на OPNsense

1. Перейдіть у **Firewall → NAT → Destination NAT**
2. Натисніть **+ Add**
3. Заповніть параметри:

| Параметр | Значення |
|---|---|
| Interface | `WAN` |
| Protocol | `TCP` |
| Destination | `WAN address` |
| Destination port range | `9443` |
| Redirect target IP | `192.168.1.XXX` |
| Redirect target port | `9443` |
| Description | `Portainer Web UI` |

![Налаштування Port Forwarding для Portainer](<image/Знімок екрана 2026-03-08 о 23.59.00.png>)

4. Натисніть **Save**, потім **Apply Changes**

> 🌐 Після налаштування Portainer буде доступний ззовні за адресою: `https://<WAN_IP>:9443`

### Перший вхід до Portainer

Під час першого входу вам буде запропоновано створити адміністратора. Введіть логін та пароль двічі:

- **Login:** `admin`
- **Password:** `PortainerStudent@`

![Створення користувача Portainer](<image/Знімок екрана 2026-03-08 о 23.02.17.png>)

### Можливі проблеми з доступом

Якщо після входу виникає помилка (як на зображенні нижче) — просто перезапустіть контейнер:

![Помилка доступу до Portainer](<image/Знімок екрана 2026-03-08 о 22.59.52.png>)

```bash
sudo docker restart portainer
```

---

## ✅ Перевірка після налаштування

- [ ] SSH підключення працює: `ssh -p 2222 student@<WAN_IP>`
- [ ] Portainer відкривається у браузері: `https://<WAN_IP>:9443`
- [ ] UFW активний і показує коректні правила: `sudo ufw status verbose`
- [ ] Docker контейнер запущено: `docker ps`

---

## 📋 Підсумок кроків

| # | Дія | Де |
|---|---|---|
| 1 | Port Forward SSH (`2222` → `22`) | OPNsense → Firewall → NAT |
| 2 | Перевірка та запуск SSH | Ubuntu Terminal |
| 3 | Підключення по SSH | Зовнішня машина |
| 4 | Встановлення Docker + Portainer | Ubuntu Terminal |
| 5 | Налаштування UFW | Ubuntu Terminal |
| 6 | Port Forward Portainer (`9443` → `9443`) | OPNsense → Firewall → NAT |