**OWASP Juice Shop** — це один із найкращих сучасних інструментів для вивчення веб-безпеки. Це навмисно вразливий додаток, написаний на Node.js, Express та Angular. Оскільки він містить понад 100 челенджів, цей посібник допоможе вам пройти перші ключові етапи та зрозуміти логіку гри.

### 1. Встановлення
1. Встановіть докер. https://www.docker.com/?_gl=1%2Ar6ej1n%2A_gcl_au%2AMTE3MDMwMTAyLjE3ODUzNTQxMDg.%2A_ga%2AOTY5NzI5ODAyLjE3ODUzNTQxMDg.%2A_ga_XJWPQMJYHQ%2AczE3ODUzOTcxNDckbzMkZzEkdDE3ODUzOTcxNDkkajU4JGwwJGgw
2. Виконайте команду в терміналі ```
```
docker pull bkimminich/juice-shop
```
![[OWASP Juice Shop-1785397588133.webp|331]]

3. Виконайте 
```
docker run --rm -p 3000:3000 bkimminich/juice-shop
```
![[OWASP Juice Shop-1785397692048.webp|456]]

4. Відкрийте браузер за адресою: **[http://localhost:3000](http://localhost:3000/)**
![[OWASP Juice Shop-1785397770930.webp|556]]

### 2. Пошук дошки результатів (Score Board)
У Juice Shop посилання на список завдань приховане. Ваше перше завдання — знайти його самостійно.

- **Завдання:** Знайти сторінку зі списком усіх челенджів.
- **Як пройти:** Спробуйте вгадати URL (наприклад, перевірте `/score-board` або `/scoreboard`). Розробники часто ховають такі технічні сторінки в коді або через роутинг. Зазвичай багато цікавого можна знайти у файлі `main.js`. Тобто натискаємо F12, відкривається інспектор і далі шукаємо в коді щось схоже на **scoreboard**. Отримуємо наступний результат:
![[OWASP Juice Shop-1785398357993.webp]]

1. Вставляємо посилання
![[OWASP Juice Shop-1785398428517.webp]]

### Базові вразливості

#### Атака на пошуковий рядок (DOM XSS)
**Завдання:** Виконати XSS-атаку.
![[OWASP Juice Shop-1785398518602.webp|327]]

 У полі пошуку (Search) вгорі сторінки введіть скрипт: 
 `<iframe src="javascript:alert('xss')">`
![[OWASP Juice Shop-1785399663181.webp|295x166]]


 Знаходимо рядок коду з вразливістю
 ![[OWASP Juice Shop-1785399753101.webp]]
Фіксимо його
![[OWASP Juice Shop-1785399884215.webp]]

**Виконуємо також Payload** 
![[OWASP Juice Shop-1785399977730.webp|302]]

Вставляємо в пошукове поле 
```
<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>

```
![[OWASP Juice Shop-1785400089958.webp]]

Також знаходимо помилку в коді та фіксимо її

![[OWASP Juice Shop-1785400160036.webp|399x217]]
![[OWASP Juice Shop-1785400180889.webp|406x228]]

#### Витік конфіденційних даних (Sensitive Data Exposure)

- **Завдання:** Знайти файл, який не має бути публічним. Спочатку шукаємо відкриті директорії, але отримуємо помилку:
```
gobuster dir -u http://127.0.0.1:3000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

```

![[OWASP Juice Shop-1785400548960.webp]]

Вносимо зміни, які нам рекомендовано, та знову запускаємо сканування:

```
 gobuster dir -u http://127.0.0.1:3000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --exclude-length 9903 

```
![[OWASP Juice Shop-1785400789724.webp]]

Перезпускаємо контейнер та йдемо до знайденого FTP:
**[http://localhost:3000/ftp](http://localhost:3000/ftp)**
![[OWASP Juice Shop-1785401171056.webp]]

Копіюємо посилання на coupons http://localhost:3000/ftp/coupons_2013.md.bak та вставляяємо його зверху.
Пробуйєм завантажити файли, які сервер намагається блокувати, використовуючи обхід фільтрів (наприклад, додаючи `%2500.md` до назви).

![[OWASP Juice Shop-1785401387537.webp]]

Пробуємо також з іншими файлами
![[OWASP Juice Shop-1785401526158.webp]]

Бачимо, що виконана вже частина завдань
![[OWASP Juice Shop-1785401682487.webp|509]]

---
### Злам акаунтів
#### Адмінська панель (SQL Injection)
Це класична атака, яка дозволяє увійти в систему без знання пароля.

1. Перейдіть на сторінку **Login**.
2. Введіть у поле Email: `' or 1=1--`
3. Введіть будь-який пароль.
Має виглядати ось так:
![[OWASP Juice Shop-1785401823205.webp]]

Шукаємо та фіксимо вразливості:
![[OWASP Juice Shop-1785402846618.webp|391x217]]
![[OWASP Juice Shop-1785402872655.webp|413x214]]
---

### Privacy Polici

![[OWASP Juice Shop-1785402051993.webp|264]]

В меню зліва відкриваємо Account Menu, переходимо в розділ **Privacy & Security**
та **Privacy Policy**
![[OWASP Juice Shop-1785402279629.webp|428]]

---

### Confidential Document

1. Переходимо до відкритої директорії FTP http://localhost:3000/ftp
2. Знаходимо та завантажимо конфіденційний файл
У списку файлів папки `/ftp`
- **`acquisitions.md`**
![[OWASP Juice Shop-1785402480269.webp|539]]
Шукаємо вразливості в коді та фіксмо їх:
![[OWASP Juice Shop-1785402727425.webp|369]]
![[OWASP Juice Shop-1785402763765.webp|369x194]]

---

### Exposed Metrics
Завдання **Exposed Metrics** вимагає знайти ендпоінт, який віддає системні метрики додатка для популярних систем моніторингу (наприклад, **Prometheus**).

Відкрий браузер і вставити в адресний рядок:

```
http://localhost:3000/metrics
```

![[OWASP Juice Shop-1785403264992.webp|507x334]]

Готово)

---
### Mass Dispel
це фановий UI-челендж. Його суть полягає в тому, щоб **закрити однією кнопкою одразу декілька спливаючих сповіщень**
Для виконання потрібно, щоб на екрані одночасно висіло **3 або більше** зелених сповіщень.

Потрібно затиснути Shift та закрите одне сповіщення, всі інші закриються теж.
![[OWASP Juice Shop-1785403464066.webp|479]]

---

### Outdated Allowlist

Відкриваємо Dev Tools та в пошуку шукаємо blockchain
![[OWASP Juice Shop-1785405215645.webp]]
Копіюємо посилання, яке ми знайшли та вставляємо його в нову вкладку

```
http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
```

![[OWASP Juice Shop-1785405311866.webp]]

![[OWASP Juice Shop-1785405466773.webp|474]]
![[OWASP Juice Shop-1785405489636.webp|478]]

---
### Repetitive Registration
Відкриваємо dev tools console та вставляємо цей код:
JavaScript

```
fetch('/api/Users/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'dry_test_' + Date.now() + '@juice-sh.op',
    password: 'Password123!',
    passwordRepeat: 'WRONG_PASSWORD!',
    securityQuestion: {
      id: 1,
      question: "Your eldest sibling's middle name?",
      createdAt: "2021-01-01",
      updatedAt: "2021-01-01"
    },
    securityAnswer: 'test'
  })
})
.then(res => res.json())
.then(data => console.log('Response:', data));
```
Ми відправляємо на бекенд поле `password` зі значенням `Password123!`, а `passwordRepeat` з **іншим** значенням `WRONG_PASSWORD!`.

Оскільки бекенд Juice Shop не валідує збіг цих полів (покладаючись лише на Angular на фронтенді), він створить користувача і відправить плашка про виконання челенджу **Repetitive Registration**!

---

### Web3 Sandbox
Методом підбору, на основі типових імен каталогів ми підбираємо таку адресу та вставляємо її в нове вікно http://127.0.0.1:3000/#/web3-sandbox
![[OWASP Juice Shop-1785407136738.webp|451]]


---
### Admin Section
За допомогою dev tools в браузері  проаналізовано файл `main.js`. У цьому файлі є визначення різних маршрутів додатка, зокрема маршрут для адміністративного розділу ("administration").

![[OWASP Juice Shop-1785407429071.webp]]

Переходимо за шляхом 127.0.0.1:3000/#/administration

![[OWASP Juice Shop-1785407540966.webp|510]]
Проте в нас недостатньо доступів

**Отримуємо адіміністативний доступ**
Входимо в систему під логіном та паролем адміна та повторно переходимо за шляхом.
![[OWASP Juice Shop-1785407871231.webp]]

---
