# 🔧 Лабораторна робота: Віддалений доступ, Docker та Portainer за OPNsense

## Топологія мережі

```
[Зовнішня мережа] → [OPNsense WAN] → [OPNsense LAN 192.168.1.1] → [Ubuntu Server 192.168.1.100]
```

| Пристрій | Роль | IP-адреса |
|---|---|---|
| OPNsense | Шлюз / Фаєрвол | LAN: `192.168.1.1` |
| Ubuntu Server | Сервер додатків | `192.168.1.1XX` |
![alt text](<image/Знімок екрана 2026-03-08 о 22.47.51.png>)
### Зверніть увагу на Ір адресу Ubuntu Server вона може юуьти в вас інша.
---

## Крок 1. Налаштування Port Forwarding для SSH на OPNsense

### 1.1 Створення правила NAT Port Forward (Firewall: NAT: Destination NAT)


1. Перейдіть у **Firewall → NAT → Destination NAT (Port Forward)**
2. Натисніть кнопку **+ Add**
3. Заповніть параметри:

| Параметр | Значення |
|---|---|
| Interface | `WAN` |
| Protocol | `TCP` |
| Destination | `WAN address` |
| Destination port range | `2222` |
| Redirect target IP | `192.168.1.100` |
| Redirect target port | `22` |
| Description | `SSH to Ubuntu` |

### Зверніть увагу що у полі  ***Redirect Target IP*** ви вказуєте ір свого серверу а не той що на зображенні знизу
![alt text](<image/Знімок екрана 2026-03-08 о 23.13.55.png>)
4. Натисніть **Save**, потім **Apply Changes**

> ⚠️ **Важливо:** OPNsense автоматично створить відповідне правило у **Firewall → Rules → WAN**. Перевірте його наявність після збереження.

---

## Крок 2. Перевірка та запуск SSH на Ubuntu

Підключіться до Ubuntu через консоль і виконайте:

```bash
# Перевірити статус SSH
sudo systemctl status ssh

# Якщо сервіс не встановлений — встановити
sudo apt update && sudo apt install openssh-server -y

# Перевірити що SSH слухає на порту 22
ss -tlnp | grep 22
```
### Зверніть увагу що коли ви будете намагатися встановити SSH в вас мже винкнути помилка як на зображенні. Щоб обійти цю проблему перезавантажте Ubuntu ввів ши команду ***reboot*** та спробуйте повторно. 
![alt text](<image/error_ssh.png>)

### перевірка стану SSH. якщо показує активний та запушений то переходемо до кроку 3. Якщо не активний то виконуєм команди знизу
```bash
# Запустити сервіс
sudo systemctl start ssh

# Додати до автозапуску
sudo systemctl enable ssh

# Перевірити що SSH слухає на порту 22
ss -tlnp | grep 22
```
![alt text](<image/Знімок екрана 2026-03-08 о 22.40.39.png>)
---

## Крок 3. Підключення через SSH з зовнішньої мережі

```bash
# Формат команди
ssh -p <зовнішній_порт> <користувач>@<WAN_IP_OPNsense>

# Приклад
ssh -p 2222 student@192.168.88.XXX
```
### Увага 192.168.88.XXX це ір адресе WAN порту вашого opnsense за яким ви підключаєтеся до веб інтефесу opnsense
> 💡 Використання нестандартного порту `2222` замість `22` зменшує кількість автоматизованих атак на SSH.
![alt text](<image/Знімок екрана 2026-03-08 о 22.42.32.png>)
---
## Крок 4. Встановлення Docker та Portainer

### 4.1 Встановлення Docker

```bash
# Додаємо офіційний GPG ключ Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Додаємо репозиторій саме для bionic
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  bionic stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Оновлюємо списки та встановлюємо тільки основні компоненти
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
![alt text](<image/isntallDocker.png>)

### 4.2 Запуск Portainer Community Edition

```bash
# Створити persistent volume для Portainer
docker volume create portainer_data

# Запустити контейнер Portainer
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

# Перевірити що контейнер запущено
docker ps
```

> 🌐 Portainer буде доступний локально за адресою: `https://192.168.1.XXX:9443`
### Увага 192.168.88.XXX це ір адресе WAN порту вашого opnsense за яким ви підключаєтеся до веб інтефесу opnsense
---

## Крок 5. Налаштування UFW на Ubuntu

> ⚠️ **Завжди** дозволяйте SSH **до** увімкнення UFW, інакше втратите доступ до сервера!

```bash
# 1. Дозволити SSH (щоб не втратити доступ!)
sudo ufw allow from 192.168.1.1 to any port 22 proto tcp

# 2. Дозволити доступ до Portainer ТІЛЬКИ з IP шлюзу OPNsense
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

---
### Створення політики на ufw для дозволу на ssh тілки з основного шлюза 
![alt text](<image/Allow_22.png>)
### Створення політики на ufw для дозволу на Portainer тілки з основного шлюза 
![alt text](<image/Allow_9443.png>)

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
![alt text](<image/Знімок екрана 2026-03-08 о 23.59.00.png>)

4. Натисніть **Save**, потім **Apply Changes**

> 🌐 Після налаштування Portainer буде доступний ззовні за адресою: `https://<WAN_IP>:9443`

![alt text](<image/Знімок екрана 2026-03-08 о 23.02.17.png>)
### Заповнюєте поле де вводите два рази пароль це вистворюєте користувача для Portainer логін ***admin*** парль два рази ***PortainerStudent@***

### можливі проблеми з доступо як на зображенні знизу 
![alt text](<image/Знімок екрана 2026-03-08 о 22.59.52.png>)
### просто перезавантажте контейнер 
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
