Ось готовий файл `README.md`, який ти можеш використовувати для своєї лабораторної роботи. Він структурований, зрозумілий і містить усі необхідні команди та параметри.

---

# Лабораторна робота: Налаштування тегованих VLAN та Docker macvlan

Ця лабораторна робота присвячена конфігурації мережевої інфраструктури на базі Ubuntu, Docker та OPNsense для ізоляції сервісів у різних VLAN-сегментах.

## Мета роботи
1. Налаштувати підтримку 802.1Q (VLAN) на хостовій ОС Ubuntu.
2. Створити віртуальні мережі в Docker за допомогою драйвера `macvlan`.
3. Розгорнути веб-сервіс (Nginx) у конкретному VLAN через Portainer.
4. Налаштувати доступ до сервісу із зовнішньої мережі через OPNsense.

---

## Етап 1: Підготовка мережевих інтерфейсів (L2)

На цьому етапі ми створюємо віртуальні інтерфейси на хості, які будуть "слухати" тегований трафік з фізичного порту `ens18`.

```bash
# 1. Активуємо модуль ядра для роботи з 802.1Q
sudo modprobe 8021q

# 2. Створюємо інтерфейси для VLAN 10, 20 та 30
sudo ip link add link ens18 name ens18.10 type vlan id 10
sudo ip link add link ens18 name ens18.20 type vlan id 20
sudo ip link add link ens18 name ens18.30 type vlan id 30

# 3. Піднімаємо створені інтерфейси
sudo ip link set dev ens18.10 up
sudo ip link set dev ens18.20 up
sudo ip link set dev ens18.30 up

# 4. Перевірка результату
ip link show | grep ens18
```

---

## Етап 2: Створення мереж у Docker (macvlan)

Драйвер `macvlan` дозволяє контейнерам мати прямий доступ до фізичної мережі через створені нами інтерфейси.

```bash
# Створення мережі для VLAN 10
sudo docker network create -d macvlan \
  --subnet=10.10.10.0/24 \
  --gateway=10.10.10.1 \
  -o parent=ens18.10 vlan10_net

# Створення мережі для VLAN 20
sudo docker network create -d macvlan \
  --subnet=10.10.20.0/24 \
  --gateway=10.10.20.1 \
  -o parent=ens18.20 vlan20_net

# Створення мережі для VLAN 30
sudo docker network create -d macvlan \
  --subnet=10.10.30.0/24 \
  --gateway=10.10.30.1 \
  -o parent=ens18.30 vlan30_net
```

---

## Етап 3: Розгортання стеку через Portainer

Для запуску контейнера використовуйте наступний Docker Compose (YAML) у розділі **Stacks**:

```yaml
version: '3.8'

services:
  nginx-server:
    image: nginx:latest
    container_name: nginx_app
    networks:
      vlan10_net:
        ipv4_address: 10.10.10.50
    restart: always

networks:
  vlan10_net:
    external: true
```

---
![alt text](<image/image1.png>)
потім натискаєте deploy the stak
![alt text](<image/image2.png>)
![alt text](<image/image3.png>)
![alt text](<image/image4.png>)
## Етап 4: Налаштування NAT на OPNsense

Щоб сервіс був доступний зовні, необхідно створити правило **Destination NAT** у веб-інтерфейсі OPNsense (**Firewall: NAT: Port Forward**):

| Параметр | Значення |
| :--- | :--- |
| **Interface** | WAN |
| **Protocol** | TCP |
| **Destination** | WAN address |
| **Destination Port** | 8888 |
| **Redirect Target IP** | 10.10.10.50 |
| **Redirect Target Port** | 80 |
| **Filter Rule Association** | Add associated filter rule |

---

## Перевірка працездатності
1. Перевірте статус інтерфейсів на Ubuntu: `ip a`.
2. Переконайтеся, що контейнер запущений у Portainer і має IP `10.10.10.50`.
3. Спробуйте відкрити сторінку Nginx за вашою зовнішньою IP-адресою: `http://[WAN_IP]:8888`.

> **Примітка:** Оскільки ми налаштовували інтерфейси через команду `ip link`, після перезавантаження Ubuntu їх потрібно буде створити заново (якщо не налаштовано Netplan).

---

Чи хочеш, щоб я додав у цей файл розділ з описом того, як автоматизувати ці дії через скрипт `.sh` для студентів?