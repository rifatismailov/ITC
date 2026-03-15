# 🔐 Лабораторна робота

## Контроль доступу та Inter-VLAN маршрутизація в DMZ

---

# 📚 Мета роботи

Навчитись:

* сегментувати мережу за допомогою **VLAN**
* налаштовувати **Inter-VLAN маршрутизацію**
* використовувати **Cisco IR800** як gateway
* впроваджувати політики безпеки за допомогою **ACL (Access Control Lists)**

---

# 🌐 Топологія мережі

| VLAN    | Призначення         | Мережа          |
| ------- | ------------------- | --------------- |
| VLAN 10 | Мережа персоналу    | 192.168.10.0/24 |
| VLAN 20 | Мережа менеджменту  | 192.168.20.0/24 |
| VLAN 30 | DMZ (серверна зона) | 192.168.30.0/24 |

---

# ⚙️ Крок 1 — Налаштування Switch2 (Access Switch)

Необхідно створити VLAN та підготувати trunk-з'єднання до роутера.

```cisco
Switch2(config)# vlan 30
Switch2(config-vlan)# name DMZ_ZONE
Switch2(config-vlan)# exit

! Налаштування порту для сервера
Switch2(config)# interface fastEthernet 0/3
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 30
Switch2(config-if)# no shutdown

! Trunk до роутера
Switch2(config)# interface fastEthernet 0/23
Switch2(config-if)# switchport mode trunk
Switch2(config-if)# switchport trunk allowed vlan 10,20,30
```

### 📖 Пояснення

**vlan 30**

Створюється окремий широкомовний домен.

**switchport mode access**

Порт використовується для кінцевого пристрою.

**switchport trunk allowed vlan**

Через trunk передаються лише дозволені VLAN.

---

# 💻 Крок 2 — Налаштування кінцевих пристроїв

## Сервер (DMZ)

| Параметр        | Значення      |
| --------------- | ------------- |
| IP Address      | 192.168.30.10 |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.30.1  |

---

## ПК VLAN 10

| Параметр    | Значення      |
| ----------- | ------------- |
| IP Address  | 192.168.10.x  |
| Subnet Mask | 255.255.255.0 |
| Gateway     | 192.168.10.1  |

---

## ПК VLAN 20

| Параметр    | Значення      |
| ----------- | ------------- |
| IP Address  | 192.168.20.x  |
| Subnet Mask | 255.255.255.0 |
| Gateway     | 192.168.20.1  |

---

# 🛠 Крок 3 — Налаштування Router IR800

IR800 використовує **SVI (Switch Virtual Interface)**.

### Trunk інтерфейс

```cisco
Router1(config)# interface gigabitEthernet 1
Router1(config-if)# switchport mode trunk
Router1(config-if)# switchport trunk allowed vlan 10,20,30
Router1(config-if)# no shutdown
```

---

### Gateway для VLAN

```cisco
Router1(config)# interface vlan 10
Router1(config-if)# ip address 192.168.10.1 255.255.255.0
Router1(config-if)# no shutdown

Router1(config)# interface vlan 20
Router1(config-if)# ip address 192.168.20.1 255.255.255.0
Router1(config-if)# no shutdown

Router1(config)# interface vlan 30
Router1(config-if)# ip address 192.168.30.1 255.255.255.0
Router1(config-if)# no shutdown
```

---

### Увімкнення маршрутизації

```cisco
Router1(config)# ip routing
```

---

# 🛡 Крок 4 — Контроль доступу (ACL)

Реалізація принципу **Least Privilege**.

```cisco
Router1(config)# ip access-list extended DMZ_POLICY

permit icmp 192.168.30.0 0.0.0.255 any echo-reply

permit tcp 192.168.30.0 0.0.0.255 any established

deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255

deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255

permit ip any any
```

---

### Прив'язка ACL

```cisco
Router1(config)# interface vlan 30
Router1(config-if)# ip access-group DMZ_POLICY in
```

---

# 📖 Пояснення правил ACL

### 1️⃣ ICMP Echo Reply

```
permit icmp ... echo-reply
```

Дозволяє серверу **відповідати на ping**.

---

### 2️⃣ TCP Established

```
permit tcp ... established
```

Дозволяє тільки **відповідь сервера на існуюче з'єднання**.

---

### 3️⃣ Заборона доступу до внутрішніх мереж

```
deny ip DMZ -> VLAN10
deny ip DMZ -> VLAN20
```

Сервер **не може ініціювати з'єднання у внутрішню мережу**.

---

### 4️⃣ Default allow

```
permit ip any any
```

Фінальне правило щоб не блокувати інший трафік.

---

# 🧠 Теорія

## Access Control

Ця лабораторна демонструє кілька моделей контролю доступу.

### DAC (Discretionary Access Control)

Адміністратор визначає політику доступу.

---

### RBAC (Role Based Access Control)

Сегментація за ролями:

* Staff
* Management
* DMZ

---

### Least Privilege

Сервер має **мінімальні дозволи** для роботи.

---

# 🧪 Завдання для перевірки

Перевірити роботу політик:

### ✔ Ping з ПК → сервер

Повинен **працювати**.

---

### ❌ Ping з сервера → ПК

Повинен **не працювати**.

---

### ❓ Питання

Чому правило

```
permit ip any any
```

стоїть **останнім у списку ACL**?

---

# 🚀 Додаткові завдання

### 1️⃣ Time-based ACL

Дозволити доступ до сервера лише в робочі години.

---

### 2️⃣ Logging атак

```cisco
deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255 log
```

Router буде записувати спроби доступу.

---

### 3️⃣ Anti-spoofing ACL

Дозволяти пакети у VLAN 10 тільки з:

```
192.168.10.0/24
```

---

# ✅ Результат

Було реалізовано:

* VLAN сегментацію
* Inter-VLAN routing
* DMZ зону
* ACL політику доступу
* базову модель мережевої безпеки

---

# 🎉 Висновок

Було створено **захищену DMZ-зону на обладнанні Cisco**, що дозволяє контролювати доступ між сегментами мережі та мінімізувати ризики несанкціонованого доступу.
