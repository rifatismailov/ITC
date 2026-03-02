# Лабораторна робота: VLAN сегментація в Cisco Packet Tracer

## Мета
Налаштувати два комутатори з VLAN 10 та VLAN 20, з'єднані trunk портом. Пристрої в одному VLAN можуть пінгувати один одного, але ізольовані від іншого VLAN.

---

## Топологія

```
VLAN 10: 192.168.10.0/24
VLAN 20: 192.168.20.0/24

    PC1 (VLAN 10)         PC3 (VLAN 10)
    192.168.10.10         192.168.10.30
         |                     |
      Fa0/1                 Fa0/1
         |                     |
    [Switch1] -------- [Switch2]
         |     Trunk      |
      Fa0/2   Fa0/24   Fa0/2
         |     VLAN      |
         |    10,20      |
    PC2 (VLAN 20)     PC4 (VLAN 20)
    192.168.20.20     192.168.20.40
```

---

## Крок 1: Створення топології в Packet Tracer

1. **Додайте пристрої:**
   - 2 × Switch 2960 (або будь-який managed switch)
   - 4 × PC

2. **З'єднайте пристрої:**
   - PC1 → Switch1 FastEthernet 0/1
   - PC2 → Switch1 FastEthernet 0/2
   - PC3 → Switch2 FastEthernet 0/1
   - PC4 → Switch2 FastEthernet 0/2
   - Switch1 Fa0/24 → Switch2 Fa0/24 (trunk)

---

## Крок 2: Налаштування IP адрес на ПК

### PC1 (VLAN 10)
```
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Gateway: (залишити порожнім, не потрібен для L2)
```

### PC2 (VLAN 20)
```
IP Address: 192.168.20.20
Subnet Mask: 255.255.255.0
Gateway: (залишити порожнім)
```

### PC3 (VLAN 10)
```
IP Address: 192.168.10.30
Subnet Mask: 255.255.255.0
Gateway: (залишити порожнім)
```

### PC4 (VLAN 20)
```
IP Address: 192.168.20.40
Subnet Mask: 255.255.255.0
Gateway: (залишити порожнім)
```

---

## Крок 3: Налаштування Switch1

```cisco
Switch1> enable
Switch1# configure terminal

! Створюємо VLAN 10 та VLAN 20
Switch1(config)# vlan 10
Switch1(config-vlan)# name DATA_VLAN
Switch1(config-vlan)# exit

Switch1(config)# vlan 20
Switch1(config-vlan)# name VOICE_VLAN
Switch1(config-vlan)# exit

! Налаштовуємо порт Fa0/1 для PC1 (VLAN 10)
Switch1(config)# interface fastEthernet 0/1
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 10
Switch1(config-if)# no shutdown
Switch1(config-if)# exit

! Налаштовуємо порт Fa0/2 для PC2 (VLAN 20)
Switch1(config)# interface fastEthernet 0/2
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 20
Switch1(config-if)# no shutdown
Switch1(config-if)# exit

! Налаштовуємо trunk порт Fa0/24 до Switch2
Switch1(config)# interface fastEthernet 0/24
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# switchport trunk allowed vlan 10,20
Switch1(config-if)# no shutdown
Switch1(config-if)# exit

Switch1(config)# exit
Switch1# write memory
```

---

## Крок 4: Налаштування Switch2

```cisco
Switch2> enable
Switch2# configure terminal

! Створюємо VLAN 10 та VLAN 20
Switch2(config)# vlan 10
Switch2(config-vlan)# name DATA_VLAN
Switch2(config-vlan)# exit

Switch2(config)# vlan 20
Switch2(config-vlan)# name VOICE_VLAN
Switch2(config-vlan)# exit

! Налаштовуємо порт Fa0/1 для PC3 (VLAN 10)
Switch2(config)# interface fastEthernet 0/1
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 10
Switch2(config-if)# no shutdown
Switch2(config-if)# exit

! Налаштовуємо порт Fa0/2 для PC4 (VLAN 20)
Switch2(config)# interface fastEthernet 0/2
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 20
Switch2(config-if)# no shutdown
Switch2(config-if)# exit

! Налаштовуємо trunk порт Fa0/24 до Switch1
Switch2(config)# interface fastEthernet 0/24
Switch2(config-if)# switchport mode trunk
Switch2(config-if)# switchport trunk allowed vlan 10,20
Switch2(config-if)# no shutdown
Switch2(config-if)# exit

Switch2(config)# exit
Switch2# write memory
```

---

## Крок 5: Перевірка конфігурації

### На Switch1 та Switch2 виконайте:

```cisco
! Перевірка створених VLAN
Switch# show vlan brief

! Перевірка trunk порту
Switch# show interfaces trunk

! Перевірка статусу портів
Switch# show interfaces status
```

**Очікувані результати:**
- VLAN 10 та VLAN 20 створені
- Fa0/1 → VLAN 10 (access mode)
- Fa0/2 → VLAN 20 (access mode)
- Fa0/24 → Trunk (VLAN 10,20 allowed)

---

## Крок 6: Тестування зв'язку (ping)

### Тест 1: Пінг всередині VLAN 10
```
PC1 (192.168.10.10) → ping 192.168.10.30 (PC3)
Результат: ✅ SUCCESS (пакети проходять через trunk)
```

### Тест 2: Пінг всередині VLAN 20
```
PC2 (192.168.20.20) → ping 192.168.20.40 (PC4)
Результат: ✅ SUCCESS (пакети проходять через trunk)
```

### Тест 3: Пінг між різними VLAN
```
PC1 (192.168.10.10) → ping 192.168.20.20 (PC2)
Результат: ❌ FAIL (Request timed out — VLAN ізольовані!)
```

```
PC3 (192.168.10.30) → ping 192.168.20.40 (PC4)
Результат: ❌ FAIL (Request timed out)
```

---

## Пояснення

### Як це працює:

1. **Access порти** (Fa0/1, Fa0/2):
   - Тегують всі пакети від ПК у відповідний VLAN (10 або 20)
   - ПК не знають про існування VLAN — це прозоро для них

2. **Trunk порт** (Fa0/24):
   - Передає пакети з тегами VLAN між комутаторами
   - Використовує протокол IEEE 802.1Q (dot1q)
   - Дозволяє передачу кількох VLAN по одному кабелю

3. **Ізоляція**:
   - VLAN 10 та VLAN 20 — це окремі broadcast domains
   - Пакети з VLAN 10 ніколи не потраплять у VLAN 20 (і навпаки)
   - Для зв'язку між VLAN потрібен Layer 3 маршрутизатор (router або L3 switch)

---

## Додаткові команди для діагностики

```cisco
! Показати MAC адреси на кожному VLAN
Switch# show mac address-table vlan 10
Switch# show mac address-table vlan 20

! Показати детальну інформацію про trunk
Switch# show interfaces fastEthernet 0/24 switchport

! Показати інформацію про VLAN
Switch# show vlan id 10
Switch# show vlan id 20
```

---

## Типові помилки та їх виправлення

### Проблема 1: PC1 не може пінгувати PC3 (обидва в VLAN 10)

**Причина:** Trunk порт не налаштовано або VLAN не дозволено на trunk

**Рішення:**
```cisco
Switch1(config)# interface fastEthernet 0/24
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# switchport trunk allowed vlan 10,20
```

### Проблема 2: Всі ПК можуть пінгувати один одного (немає ізоляції)

**Причина:** Порти не призначено до VLAN або всі в VLAN 1 (default)

**Рішення:**
```cisco
! Перевірте
Switch# show vlan brief

! Якщо всі порти в VLAN 1, переконфігуруйте:
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport access vlan 10
```

### Проблема 3: Trunk порт показує "err-disabled"

**Причина:** DTP (Dynamic Trunking Protocol) конфлікт

**Рішення:**
```cisco
Switch(config)# interface fastEthernet 0/24
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
! Або вимкніть DTP:
Switch(config-if)# switchport nonegotiate
```

---

## Розширення завдання (опціонально)

### Додайте VLAN 30 з новими ПК
```cisco
! На обох комутаторах:
Switch(config)# vlan 30
Switch(config-vlan)# name GUEST_VLAN
Switch(config-vlan)# exit

! Додайте порт у VLAN 30:
Switch(config)# interface fastEthernet 0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 30
Switch(config-if)# exit

! Додайте VLAN 30 до trunk:
Switch(config)# interface fastEthernet 0/24
Switch(config-if)# switchport trunk allowed vlan add 30
```

---

## Контрольні питання

1. Що таке trunk порт і навіщо він потрібен?
2. Чи можуть ПК в різних VLAN спілкуватись без маршрутизатора?
3. Що станеться, якщо видалити VLAN 10 з trunk allowed list?
4. Яка різниця між `switchport mode access` та `switchport mode trunk`?

---

## Звіт

Для здачі роботи підготуйте:
1. Скріншот топології в Packet Tracer
2. Скріншот `show vlan brief` з Switch1
3. Скріншот `show interfaces trunk` з Switch1
4. Скріншот успішного ping між PC1 → PC3 (VLAN 10)
5. Скріншот неуспішного ping між PC1 → PC2 (різні VLAN)
6. Короткі відповіді на контрольні питання

---

**Вітаю! Ви налаштували VLAN сегментацію в Cisco мережі! 🎯**
