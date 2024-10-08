# Лабораторная работа №1. Основы HTTP

## Задание №1. Анализ HTTP-запросов

1. Откройте веб-браузер и зайдите на сайт [http://sandbox.usm.md/login](http://sandbox.usm.md/login).

   В адресную строку ввел URL сайта.

   ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/01.png>)

2. Откройте вкладку `Network` в инструментах разработчика браузера.

   С помощью комбинаций клавиш `Command + F` открываю панель разработчиков. Далее перехожу во вкладку `Network`.

   ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/02.png>)

3. Введите неверные данные для входа (например, `username: student`, `password: studentpass`).

   Ввожу неверные данные в форму.

   ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/03.png>)

4. Проанализируйте запросы, которые были отправлены на сервер.

   Запрос завершился с ошибкой `401 Unauthorized`. Перейдя во вкладку `Payload` увидим тело запроса, которые были отправлены на сервер.

5. Ответьте на следующие вопросы:

   - Какой метод HTTP был использован для отправки запроса?

     Для отправки запроса был использован метод `POST`

   - Какие заголовки были отправлены в запросе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/04.png>)

   - Какие параметры были отправлены в запросе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/05.png>)

   - Какой код состояния был возвращен сервером?

     Сервером был возвращен ответ `Status Code: 401 Unauthorized`

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/06.png>)

   - Какие заголовки были отправлены в ответе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/07.png>)

6. Повторите шаги 3-5, введя верные данные для входа (`username: admin`, `password: password`).

7. Введите верные данные для входа (например, `username: admin`, `password: password`).

   Ввожу верные данные в форму.

   ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/08.png>)

8. Проанализируйте запросы, которые были отправлены на сервер.

   Запрос завершился с удачно `200 OK`. Перейдя во вкладку `Payload` увидим тело запроса, которые были отправлены на сервер.

9. Ответьте на следующие вопросы:

   - Какой метод HTTP был использован для отправки запроса?

     Для отправки запроса был использован метод `POST`

   - Какие заголовки были отправлены в запросе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/09.png>)

   - Какие параметры были отправлены в запросе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/10.png>)

   - Какой код состояния был возвращен сервером?

     Сервером был возвращен ответ `Status Code: 200 OK`

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/11.png>)

   - Какие заголовки были отправлены в ответе?

     ![Скриншот страницы](<../Lab Work 1 - Basics of HTTP/images/12.png>)

## Задание №2. Составление HTTP-запросов

1. Составьте `GET`-запрос к серверу по адресу `http://sandbox.com`, указав в заголовке `User-Agent` ваше имя и фамилию.

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/13.png>)

2. Составьте `POST`-запрос к серверу по адресу `http://sandbox.com/cars`, указав в теле запроса следующие параметры:

   ```json
   {
     "make": "Toyota",
     "model": "Corolla",
     "year": 2020
   }
   ```

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/14.png>)

3. Составьте `PUT`-запрос к серверу по адресу `http://sandbox.com/cars/1`, указав в заголовке `User-Agent` ваше имя и фамилию, в заголовке `Content-Type` значение `application/json` и в теле запроса следующие параметры:

   ```json
   {
     "make": "Toyota",
     "model": "Corolla",
     "year": 2021
   }
   ```

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/15.png>)

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/16.png>)

4. Напишите один из возможных вариантов ответа сервера следующий запрос.

   ```http
   POST /cars HTTP/1.1
   Host: sandbox.com
   Content-Type: application/json
   User-Agent: John Doe
   model=Corolla&make=Toyota&year=2020
   ```

   Один из возможных вариантов ответа сервера на данный запрос

   ```json
   {
     "id": 123,
     "model": "Corolla",
     "make": "Toyota",
     "year": 2020,
     "status": "created"
   }
   ```

## Задание №3. Дополнительное задание. HTTP_Quest

Для выполнения запросов на сервер использовал `Insomnia`.

1. Отправьте `POST`-запрос на сервер по адресу `http://sandbox.usm.md/quest`, указав в заголовке User-Agent вашу фамилию и имя (Например `User-Agent: John Doe`).

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/17.png>)

2. Следуйте инструкциям на сервере, выполняя их по порядку.

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/18.png>)

3. В конце квеста Вам будет показано секретное слово, которое Вы должны будете предоставить в отчете.

   ![Скриншот экрана'](<../Lab Work 1 - Basics of HTTP/images/19.png>)

**Секретный ключ от квеста:** JyUEHxEoQzY5ExcbEwxbfQ==
