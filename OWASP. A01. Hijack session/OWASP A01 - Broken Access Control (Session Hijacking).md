**Дата:** 2026-07-25

**Середовище:** Kali Linux, Docker engine, OWASP WebGoat container.

##### Концепція:
Розробники прикладного програмного забезпечення, які створюють власні ідентифікатори сесій (session IDs), часто забувають забезпечити рівень складності та рандомізації, необхідний для безпеки. Якщо специфічний ідентифікатор сесії користувача не є складним і випадковим, додаток стає надзвичайно вразливим до атак типу «brute force» (перебір) на сесії.

##### Цілі:
Отримати доступ до автентифікованої сесії, що належить іншому користувачу.

---
##### Основні терміни:
- **Session ID** — ідентифікатор сесії.
- **Complexity and randomness** — складність та випадковість.
- **Brute force attacks** — атаки методом грубої сили (перебору).
- **Authenticated session** — автентифікована сесія (сеанс).
---
Рекомендовано перед початком пройти попередні кроки в webgoat
![[Pasted image 20260729225408.png]]

### 1. Переходимо на вкладку **(A1) Broken Access Control** та потім **Hijack a session**

 У цьому уроці ми намагаємося передбачити значення «hijack_cookie». Файл «hijack_cookie» використовується для розрізнення автентифікованих та анонімних користувачів WebGoat.
 
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785355393717.webp]]

1. Відкриваємо burpsuite в лівій панелі.


![[OWASP A01 - Broken Access Control (Session Hijacking)-1785355623143.webp|404|330x367]]
Якщо його немає то можна встановити 


одною командою в терміналі:

`sudo apt update && sudo apt install -y burpsuite

Після цього запускаємо його та переходимо на вкладку **Proxy**






Запускаємо браузер, в якому відкриваємо "WebGoat" та створюємо користавача і логінуємся або одразу логінуємось (якщо не видаляли контейнер) та переходимо на другий крок: 
Налаштування FoxyProxy у Firefox. Спочатку його потрібно встановити **FoxyProxy Standard** (у магазині add-ons). Після цього потрібно додати новий проксі. 
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785359535184.webp]]

Після цього необхідно перейти за посиланням http://burp та завантажити сертифікат.
- У Firefox зайди в **Settings** $\rightarrow$ **Privacy & Security** $\rightarrow$ прокрути вниз до **Certificates** $\rightarrow$ натисни **View Certificates**    
- У вкладці **Authorities** натисни **Import...** і вибери завантажений `cacert.der`.    
- Постави галочку **"Trust this CA to identify websites"** і натисни **OK**.
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785361471940.webp]]

Натискаємо "Access", отримуємо помилку та повертаємось до **burpsuit**, де маємо передивитись вкладинку **HTTPhistory**:
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785361535437.webp]]

Далі шукаємо відповідь **POST**, в якої міститься інформація про цю помилку (ключ - це зміст **hijack_cookie**, яку нам і потрібно "передбачити"):

![[OWASP A01 - Broken Access Control (Session Hijacking)-1785362921495.webp]]

Далі натискаємо праву кнопку миші та відправляємо запит до **Repeater**:

![[OWASP A01 - Broken Access Control (Session Hijacking)-1785363195505.webp]]

В **Reapeater** натискаємо кнопку **Send** Приблизно 10 разів

![[OWASP A01 - Broken Access Control (Session Hijacking)-1785363517720.webp]]

бачимо, як змінюється значення **hijack_cookie**, копіюємо ці значення в текстовий редактор
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785363595948.webp]]

![[OWASP A01 - Broken Access Control (Session Hijacking)-1785363848669.webp|385]]

Навіть поверхневий аналіз дає висновок про логіку створення кукі, перша частина відповідає за сесію користувача, а друга скоріш за все є **часовою міткою**, або **timestamp**.

Бачимо, що після 3286560352665800411-1785363427434 може бути 3286560352665800412-1785363427434. Тому передаємо 3286560352665800411-1785363427434 до **Intruder**.
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785364211127.webp]]

Далі в **Intruder** формуємо свій запит відповідним чином:
 Після JSESSIONID додаємо **hijack_cookie=3286560352665800412-1785363427434; **
 ![[OWASP A01 - Broken Access Control (Session Hijacking)-1785364595670.webp]]

Натискаємо **Start attack**
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785365302089.webp]]
Переходимо назад у браузер, бачимо, що завдання виконане
![[OWASP A01 - Broken Access Control (Session Hijacking)-1785365349325.webp]]