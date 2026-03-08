# Лабораторна робота: Основи криптографії

## Мета
Практично ознайомитись з основними криптографічними інструментами: симетричне/асиметричне шифрування, хеш-функції, цифрові підписи.

---

## Завдання 1: Хеш-функції (SHA-256)

### 1.1. Обчислення хешу файлу

**Linux/macOS:**
```bash
# Створіть текстовий файл
echo "Hello, World!" > message.txt

# Обчисліть SHA-256 хеш
sha256sum message.txt
# Вихід: dffd6021bb2bd5b0af676290809ec3a53191dd81c7f70a4b28688a362182986f

# Змініть один символ
echo "Hello, World?" > message.txt

# Обчисліть хеш знову
sha256sum message.txt
# Вихід: ПОВНІСТЮ ІНШИЙ хеш!
```

**Windows (PowerShell):**
```powershell
# Створіть файл
Set-Content -Path message.txt -Value "Hello, World!"

# Обчисліть SHA-256
Get-FileHash message.txt -Algorithm SHA256

# Змініть файл і порівняйте
Set-Content -Path message.txt -Value "Hello, World?"
Get-FileHash message.txt -Algorithm SHA256
```

**Або використайте онлайн:**
- https://emn178.github.io/online-tools/sha256.html

### 1.2. Перевірка цілісності файлу

**Сценарій:** Ви завантажили програму з Інтернету. Як перевірити, що файл не пошкоджений?

```bash
# Припустимо, розробник опублікував офіційний хеш:
# ubuntu-22.04.iso
# SHA256: 84eed...a22b (офіційний з сайту)

# Ви завантажили файл і хочете перевірити
sha256sum ubuntu-22.04.iso

# Порівняйте хеші:
# Якщо збігаються ✅ → файл оригінальний
# Якщо НЕ збігаються ❌ → файл змінено або пошкоджено
```

**Питання для звіту:**
1. Чому зміна одного символу змінює весь хеш?
2. Чи можна відновити оригінальний текст з хешу? Чому ні?

---

## Завдання 2: Симетричне шифрування (AES)

### 2.1. Шифрування файлу за допомогою OpenSSL

**Шифрування (AES-256-CBC):**
```bash
# Створіть секретний файл
echo "Це секретне повідомлення!" > secret.txt

# Зашифруйте файл (введіть пароль при запиті)
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.txt.enc

# Перевірте — зашифрований файл нечитабельний
cat secret.txt.enc
# Вихід: бінарний "мотлох" — не можна прочитати!
```

**Розшифрування:**
```bash
# Розшифруйте файл (введіть той самий пароль)
openssl enc -aes-256-cbc -d -in secret.txt.enc -out secret_decrypted.txt

# Перевірте — файл відновлено
cat secret_decrypted.txt
# Вихід: "Це секретне повідомлення!"
```

### 2.2. Що станеться, якщо неправильний пароль?

```bash
# Спробуйте розшифрувати з іншим паролем
openssl enc -aes-256-cbc -d -in secret.txt.enc -out wrong.txt
# Введіть ІНШИЙ пароль

# Результат: Помилка або "мотлох"
# bad decrypt ... wrong final block length
```

**Питання для звіту:**
1. Який ключ використовується для шифрування та розшифрування?
2. Чому важливо зберігати пароль (ключ) у секреті?

---

## Завдання 3: Асиметричне шифрування (RSA)

### 3.1. Генерація пари ключів RSA

```bash
# Згенеруйте приватний ключ (2048 біт)
openssl genrsa -out private_key.pem 2048

# Перегляньте приватний ключ
cat private_key.pem

# Витягніть публічний ключ з приватного
openssl rsa -in private_key.pem -pubout -out public_key.pem

# Перегляньте публічний ключ
cat public_key.pem
```

**Результат:**
- `private_key.pem` — **ПРИВАТНИЙ** ключ (🔒 НЕ передавайте нікому!)
- `public_key.pem` — **ПУБЛІЧНИЙ** ключ (🔓 можна ділитися)

### 3.2. Шифрування та розшифрування

**Alice шифрує повідомлення для Bob:**

```bash
# Alice має public_key.pem Bob (публічний ключ)
echo "Привіт, Bob! Це секрет." > message.txt

# Alice шифрує повідомлення публічним ключем Bob
openssl rsautl -encrypt -pubin -inkey public_key.pem -in message.txt -out message.enc

# Зашифроване повідомлення — нечитабельне
cat message.enc
# Бінарні дані
```

**Bob розшифровує повідомлення:**

```bash
# Bob має свій приватний ключ private_key.pem
openssl rsautl -decrypt -inkey private_key.pem -in message.enc -out message_decrypted.txt

# Bob читає повідомлення
cat message_decrypted.txt
# Вихід: "Привіт, Bob! Це секрет."
```

**Питання для звіту:**
1. Чи може Charlie (третя особа) розшифрувати повідомлення, маючи тільки public_key.pem?
2. Що станеться, якщо Bob втратить свій приватний ключ?

---

## Завдання 4: Цифровий підпис

### 4.1. Alice підписує документ

```bash
# Alice створює документ
echo "Договір між Alice та Bob" > contract.txt

# Alice обчислює хеш документа
sha256sum contract.txt > contract.txt.sha256

# Alice підписує хеш своїм приватним ключем
openssl rsautl -sign -inkey private_key.pem -in contract.txt.sha256 -out contract.txt.sig

# Тепер Alice надсилає Bob:
# 1. contract.txt (документ)
# 2. contract.txt.sig (підпис)
# 3. public_key.pem (публічний ключ Alice)
```

### 4.2. Bob перевіряє підпис

```bash
# Bob отримав файли від Alice
# Bob перевіряє підпис

# Крок 1: Bob розшифровує підпис публічним ключем Alice
openssl rsautl -verify -pubin -inkey public_key.pem -in contract.txt.sig -out verified_hash.txt

# Крок 2: Bob обчислює хеш документа
sha256sum contract.txt > computed_hash.txt

# Крок 3: Bob порівнює хеші
diff verified_hash.txt computed_hash.txt

# Якщо файли ідентичні ✅ → підпис валідний (документ від Alice і не змінено)
# Якщо файли різні ❌ → підпис НЕ валідний
```

### 4.3. Що станеться, якщо документ змінено?

```bash
# Зловмисник змінює документ
echo "Договір між Alice та Charlie" > contract.txt

# Bob перевіряє підпис знову
sha256sum contract.txt > computed_hash_new.txt
diff verified_hash.txt computed_hash_new.txt

# Результат: РІЗНІ хеші ❌
# Підпис НЕ валідний → документ було змінено!
```

**Питання для звіту:**
1. Що підтверджує цифровий підпис? (3 властивості)
2. Чому важливо використовувати хеш перед підписом (а не підписувати весь документ)?

---

## Завдання 5: Хешування паролів (bcrypt)

### 5.1. Встановлення інструменту

**Linux/macOS:**
```bash
# Встановіть htpasswd (Apache utilities)
sudo apt-get install apache2-utils  # Ubuntu/Debian
# або
brew install htpasswd  # macOS
```

**Або використайте Python:**
```bash
pip install bcrypt
```

### 5.2. Створення хешу паролю

**За допомогою htpasswd:**
```bash
# Створіть хеш паролю
htpasswd -nbB user "MySecretPassword123"

# Вихід:
# user:$2y$05$... (bcrypt hash)
```

**За допомогою Python:**
```python
import bcrypt

# Пароль користувача
password = "MySecretPassword123"

# Генерація salt та хеш
salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(password.encode(), salt)

print(f"Хеш: {hashed.decode()}")

# Перевірка паролю
if bcrypt.checkpw(password.encode(), hashed):
    print("✅ Пароль правильний!")
else:
    print("❌ Пароль неправильний!")
```

### 5.3. Чому НЕ використовувати MD5 для паролів?

```bash
# Приклад з MD5 (❌ НЕ робіть так у реальності!)
echo -n "password123" | md5sum
# Вихід: 482c811da5d5b4bc6d497ffa98491e38

# Проблема: MD5 швидкий → легко brute force
# 1 GPU може перевірити 50+ мільярдів MD5 хешів/сек

# bcrypt навмисно ПОВІЛЬНИЙ (10-100 мс на 1 хеш)
# → brute force неможливий
```

**Питання для звіту:**
1. Що таке salt і навіщо він потрібен?
2. Чому bcrypt краще за MD5 для паролів?

---

## Завдання 6: TLS/SSL сертифікати (бонус)

### 6.1. Перегляд сертифіката веб-сайту

```bash
# Отримайте сертифікат Google
openssl s_client -connect google.com:443 -showcerts

# Або лише інформацію про сертифікат
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -text
```

**Що шукати:**
- **Issuer** (CA) — хто підписав сертифікат (наприклад, Google Trust Services)
- **Subject** — для кого сертифікат (google.com)
- **Validity** — термін дії (Not Before → Not After)
- **Public Key** — публічний ключ (RSA 2048 або ECC 256)

### 6.2. Створення самопідписаного сертифіката

```bash
# Генерація приватного ключа та сертифіката
openssl req -x509 -newkey rsa:2048 -keyout server_key.pem -out server_cert.pem -days 365 -nodes

# Заповніть поля:
# Country Name: UA
# Organization Name: My Company
# Common Name: localhost

# Тепер у вас є:
# server_key.pem — приватний ключ сервера
# server_cert.pem — публічний сертифікат
```

**Використання (приклад з Python HTTPS server):**
```python
import http.server
import ssl

server_address = ('localhost', 4443)
httpd = http.server.HTTPServer(server_address, http.server.SimpleHTTPRequestHandler)

# Додайте TLS
httpd.socket = ssl.wrap_socket(httpd.socket,
                                server_side=True,
                                certfile='server_cert.pem',
                                keyfile='server_key.pem',
                                ssl_version=ssl.PROTOCOL_TLS)

print("HTTPS Server running on https://localhost:4443")
httpd.serve_forever()
```

---

## Звіт

Підготуйте звіт з наступними розділами:

### 1. Хеш-функції
- Скріншот обчислення SHA-256 хешу для двох різних текстів
- Відповіді на питання (Avalanche effect, односторонність)

### 2. Симетричне шифрування
- Скріншот шифрування файлу за допомогою AES
- Скріншот успішного розшифрування
- Скріншот помилки при неправильному паролі

### 3. Асиметричне шифрування
- Скріншот згенерованих ключів (public_key.pem та private_key.pem)
- Скріншот шифрування та розшифрування повідомлення
- Відповіді на питання (чому Charlie не може розшифрувати)

### 4. Цифровий підпис
- Скріншот створення підпису
- Скріншот успішної перевірки підпису
- Скріншот невдалої перевірки (після зміни документа)
- Відповіді на питання (3 властивості підпису)

### 5. Хешування паролів
- Скріншот створення bcrypt хешу
- Пояснення (3-5 речень), чому bcrypt краще за MD5

### 6. Висновки
- Короткий підсумок (5-7 речень): що ви дізналися про криптографію, де вона застосовується в реальному житті

---

## Додаткові ресурси

**Онлайн інструменти (якщо OpenSSL недоступний):**
- SHA-256: https://emn178.github.io/online-tools/sha256.html
- AES шифрування: https://www.devglan.com/online-tools/aes-encryption-decryption
- RSA шифрування: https://www.devglan.com/online-tools/rsa-encryption-decryption

**Документація:**
- OpenSSL: https://www.openssl.org/docs/
- bcrypt: https://pypi.org/project/bcrypt/

**Безпека:**
⚠️ **УВАГА:** Не використовуйте згенеровані ключі з лабораторної для реальних систем!
Це тільки для навчання. У продакшн використовуйте:
- Довжина ключа RSA: мінімум 2048 біт (краще 4096)
- Для паролів: bcrypt, scrypt, або Argon2
- Ніколи не зберігайте приватні ключі у відкритому вигляді

---

**Успіхів у лабораторній роботі! 🔒**
