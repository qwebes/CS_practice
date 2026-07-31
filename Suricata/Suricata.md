### 1. Дізнаємося назву мережевого інтерфейсу
Вводимо в терміналі команду
```
ip a
```

 Нам потрібен той, який відповідає за вихід у мережу (зазвичай це **`eth0`**, **`wlan0`** або **`enp0s3`**).
![[Suricata-1785488361811.webp|535]]

### 2. Встановлення Suricata та утиліти

Встановлення необхідних пакетів:

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq -y
``````

![[Suricata-1785488886946.webp|628]]

Перевірка статусу служби:

```
sudo systemctl status suricata
```

![[Suricata-1785489067940.webp]]

### 3. Кастомізація  конфігурацій

```
sudo nano /etc/suricata/suricata.yaml
```

![[Suricata-1785489349661.webp|539]]

### 4. Завантаження Suricata Signatures
```
sudo suricata-update
```

![[Suricata-1785489516214.webp|658]]

Suricata використовує сигнатури для спрацьовування тривог, тому необхідно встановити їх та підтримувати актуальними. 
Сигнатури також називаються правилами, тому файлів правил. 
За допомогою інструменту suricata-update можна отримувати, оновлювати та керувати правилами, які надаються для Suricata.
### 5. Запуск Suricata

```
sudo systemctl restart suricata
```

```
sudo tail /var/log/suricata/suricata.log
```

```
sudo tail -f /var/log/suricata/stats.log
```

![[Suricata-1785489703622.webp|627]]

![[Suricata-1785489815810.webp|552]]

Цей запис журналу повідомляє про запуск движка Suricata та створення потоків різних типів. 
Конкретно, зазначено, що створено один потік для обробки пакетів (W - Workers), 
один потік для обробки вивідних пакетів (FM - Flow Managers) 
та один потік для розбирання вхідних пакетів (FR - Flow Recyclers). 
Також вказано, що движок був успішно запущений.
### 6. Опрацювання тривог разом з Suricata сигнатурами

```
sudo tail -f /var/log/suricata/fast.log
```

```
curl http://testmynids.org/uid/index.html
```
![[Suricata-1785490037055.webp|509]]

![[Suricata-1785491225430.webp]]
Suricata використовує сигнатури для спрацьовування тривог, тому необхідно встановити їх та підтримувати актуальними. 
Сигнатури також називаються правилами, тому файлів правил. 
За допомогою інструменту suricata-update можна отримувати, оновлювати та керувати правилами, які надаються для Suricata.

### 7. EVE JSON Suricata logs format

```
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="stats")'
```
![[Suricata-1785490429041.webp]]

Засіб виводу  EVE виводить сповіщення, аномалії, метадані, інформацію про файли та записи, специфічні для протоколу, у форматі JSON.

Найпоширеніший спосіб використання цього – за допомогою 'EVE', який є потоковим підходом, де всі ці логі записуються в один файл.

Щоб побачити, як це виглядає, рекомендується використовувати утиліту jq для обробки виводу у форматі JSON

### 7a. EVE JSON Suricata logs format

```
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")' 
```

![[Suricata-1785491310421.webp]]

### 8. Формат правил Suricata
**Action** - Дія, визначення того, що відбувається, коли правило виконується (знаходиться співпадіння). 

**Heade**r – Заголовок правила, що визначає протокол, IP-адреси, порти та напрямок даних.

**Rule options** - Параметри правила, що визначають контекст застосування правила.

### 9. Suricata local rules creation and configuring suricata.yaml

##### Створення кастомних правил (`local.rules`)
Відкриваємо директорію та файл у ній. Записуємо туди власні правила

```
sudo nano /var/lib/suricata/rules/local.rules
```

```

alert http any any -> any any (msg: "do not read gossip during work"; flow: to_client, established; classtype: policy-violation; sid: 10001; rev: 1;)
alert icmp any any -> any any (msg: "finally pinged"; sid: 10002; rev: 1;)

```

- **Перше правило (HTTP):** логує HTTP-трафік із повідомленням `"do not read gossip during work"` (з категорією `policy-violation` та ID `10001`).

- **Друге правило (ICMP):** логує `ping` із повідомленням `"finally pinged"` (ID `10002`).

![[Suricata-1785491740346.webp|567]]

##### Підключення цього файла в `suricata.yaml`

```
nano /etc/suricata/suricata.yaml
```

##### Додаємо `local.rules`
Просто спустися курсором клавіатури вниз і **дописами рядок `- local.rules`** під `- suricata.rules`:
```
default-rule-path: /var/lib/suricata/rules

rule-files:
  - suricata.rules
  - local.rules
```

![[Suricata-1785492380480.webp|598]]

### 10. Suricata local rules management

```
sudo suricata -c /etc/suricata/suricata.yaml -i eth0 -v
```

- **`suricata`** — запуск самого аналізатора.
    
- **`-c /etc/suricata/suricata.yaml`** — вказує шлях до конфігураційного файла, де підключені твої правила.
    
- **`-i enp0s3`** — вказує мережевий інтерфейс, який Suricata має «слухати». _(У тебе в Kali це був `eth0`, тому ти вводила `-i eth0`)_.
    
- **`-v`** — вмикає детальний вивід дій (verbose mode).

![[Suricata-1785492592190.webp]]

### 11. Suricata local rules in action

Перегляд логів у режимі реального часу за допомогою команди:

```
sudo tail -f /var/log/suricata/fast.log
```

![[Suricata-1785492707897.webp]]