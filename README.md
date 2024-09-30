# WEB-31 Долматов Фёдор Highload

## [ChatGPT](https://chatgpt.com)

## 1. Тема и целевая аудитория

### Описание

- ChatGPT - это AI-чатбот для общения, поиска информации, решения задач с помощью генеративных LLM.

### Целевая аудитория

ChatGPT имеет ~ 121.3 MAU, наибольшее число пользователей находится в Индии и США[1].

- Местоположение

| Страна         | Процент пользователей |
| -------------- | --------------------- |
| Индия          | 15.26%                |
| США            | 14.43%                |
| Великобритания | 4.67%                 |
| Бразилия       | 4.62%                 |
| Другие страны  | 61.02%                |

- Возраст

| Возраст | Процент пользователей |
| ------- | --------------------- |
| 18-24   | 23.19%                |
| 25-34   | 33.95%                |
| 35-44   | 20.65%                |
| 45-54   | 12.27%                |
| 55-64   | 6.45%                 |
| 65+     | 3.49%                 |

### MVP функционал

- Регистрация, авторизация
- Обучение модели / разметка данных
- Обработка текстовых запросов моделью
- Система оценки ответов
- История чатов
- Обработка вложений моделью

### Ключевые продуктовые решения

- Возможность обращаться к сторонним сервисам с помощью модели
- Дообучение модели (на основе фидбека)
- Монетизация (доступ к более новым версиям модели)

## 2. Расчёт нагрузки

### Продуктовые метрики [[2](#список-источников)]:

| Метрика | Значение   |
| ------- | ---------- |
| MAU     | > 100 млн. |
| DAU     | ~ 9 млн.   |

- Поскольку точные данные отсутствуют, расчет DAU был выполнен на основе предположения, что каждый пользователь активен 10-15 (~13) дней в месяц, при общем количестве ~121 млн MAU.
- Средний размер хранилища пользователя:

| Данные                              | Объем                             |
| ----------------------------------- | --------------------------------- |
| История чатов                       | ((75 _ n + (225 + 1) _ m) \* k) Б |
| Медиафайлы                          | (0,5 _ i _ k) МБ                  |
| Настройки персонализации и аккаунта | 1 МБ                              |

- Средний размер вопроса от пользователя ~ 10 слов [[3](#список-источников)] (50 символов) = 50 - 100 Б (В зависимости от языка), в среднем 75 Б
- Средний размер медиафайла при запросе ~ 0,5 МБ
- Средний размер ответа модели ~ 30 слов [[3](#список-источников)] (150 символов) = 150 - 300 Б (В зависимости от языка), в среднем 225 Б + 1 Б на информацию об оценке ответа
- Учитывая, что в одном чате может храниться несколько сообщений, как от пользователя, так и от модели, то необходимо умножить объемы, занимаемые ими, на соответствующие коэффициенты: n - количество сообщений от пользователя, m - количество ответов от модели, k - количество чатов / год.
- Аналогично с медиафайлами, которых в одном чате в среднем i штук.
- В среднем n = m = 6 (на каждый запрос один ответ), k = 156 (13 дней в месяц / год), i = 0,25
- После рассчета получаем в среднем на одного пользователя / год:

| Данные                              | Объем  |
| ----------------------------------- | ------ |
| История чатов                       | 282 КБ |
| Медиафайлы                          | 20 МБ  |
| Настройки персонализации и аккаунта | 1 МБ   |

- Средний размер хранилища модели
- По данным OpenAI [[4](#список-источников)] размер модель GPT-4 имеет ~ 1,8 трлн. параметров, при весе одного параметра ~ 4Б (float32), итоговый размер самой модели 1,8 _ 10 ^ 12 _ 4 Б ~ 7,2 ТБ

| Данные                | Объем       |
| --------------------- | ----------- |
| Обученная модель      | > 7,2 ТБ    |
| Данные для обучения   | > 950 ГБ    |
| Данные для дообучения | > 50 ГБ/мес |

- Среднее количество действий пользователя по MVP:

| Действие                               | Количество в день на одного пользователя (в среднем) |
| -------------------------------------- | ---------------------------------------------------- |
| Регистрация                            | 0,0027 (1 / год)                                     |
| Авторизация                            | 0,033 (1 / месяц)                                    |
| Открытие страницы чата                 | 2                                                    |
| Получение истории чатов                | 1                                                    |
| Получение данных пользователя          | 1                                                    |
| Отправка текстового запроса для модели | 2                                                    |
| Отправка медиа-запроса для модели      | 0,5                                                  |
| Отправка оценки на ответ модели        | 0,033                                                |

### Технические метрики

#### Хранилище:

Расчет объема хранилища на n лет:

- Данные для дообучения ~ 24 \* 10 ГБ ~ 240 ГБ
- Модели и исходные данные для их обучения ~ 1,2 ТБ
- При линейно растущем числе пользователей (10% / год) объем данных (282 КБ + 20 МБ + 1 МБ) ~ 21.3 МБ нужно умножить на 121 _ 10 ^ (1 - n) _ (11 ^ n - 10 ^ n) // хранить данные для 121 млн пользователей за n лет, 121 \* 1,1 за n - 1 лет и т.д.
- Для n = 2 получим (21,3 _ (121 + 121 _ 1,1) _ 10 ^ 6) МБ + (7,2 ТБ + 950 ГБ + 50 ГБ _ 12) \* 2 ~ **5,49 ПБ**

#### RPS

- RPS = DAU / (24 _ 60 _ 60) \* Кол-во действий в среднем на пользователя
- RPS (пик ~ 1.5х нагрузка (предположительно)):

| Действие                               | RPS   | RPS (пик) |
| -------------------------------------- | ----- | --------- |
| Регистрация                            | 0,28  | 0,42      |
| Авторизация                            | 3,44  | 5,16      |
| Открытие страницы чата                 | 104,2 | 156,3     |
| Получение медиа данных                 | 104,2 | 156,3     |
| Получение истории чатов                | 104,2 | 156,3     |
| Получение данных пользователя          | 104,2 | 156,3     |
| Отправка текстового запроса для модели | 208,4 | 312,6     |
| Отправка медиа-запроса для модели      | 52    | 78,2      |
| Отправка оценки на ответ модели        | 3,44  | 5,16      |

#### Сетевой трафик

Основной трафик приходится на открытие страницы чата, получение медиа-данных и данных пользователя, отправку текстового/медиа-запросов модели:

| Действие                               | Трафик                                                           | Трафик (пик)                                 | Трафик в сутки                       |
| -------------------------------------- | ---------------------------------------------------------------- | -------------------------------------------- | ------------------------------------ |
| Открытие страницы чата                 | 104,2 \* (1,8 КБ) ~ 187 КБ/c ~ 1,5 Мбит/c                        | 156,3 \* (1,8 КБ + 0.5 МБ) ~ 2,25 Мбит/с     | 1,5 Мбит/c \* 86400 c ~ 16,2 ГБ/сут  |
| Получение медиа данных                 | 104,2 \* (0,5 МБ) ~ 52,1 МБ/c ~ 416,8 Мбит/c                     | 156,3 \* (1,8 КБ + 0.5 МБ) ~ 625,2 Мбит/с    | 416,8 Мбит/c \* 86400 c ~ 4,5 ТБ/сут |
| Получение истории чатов                | 104,2 _ 156 _ 40 Б (размер заголовка чата) ~ 650 КБ/с ~ 5 Мбит/с | 156,3 _ 156 _ 40 Б ~ 8 Мбит/с                | 5 Мбит/с \* 86400 с ~ 54 ГБ/сут      |
| Получение данных пользователя          | 104,2 \* 1 МБ ~ 104,2 МБ/с ~ 834 Мбит/с                          | 156,3 \* 1 МБ ~ 1251 Мбит/c                  | 834 Мбит/с \* 86400 с ~ 9 ТБ/сут     |
| Отправка текстового запроса для модели | 208,4 \* (75 Б + 226 Б) ~ 63 КБ/c ~ 0,5 Мбит/c                   | 312,6 \* (75 Б + 226 Б) ~ 0,75 Мбит/с        | 0,5 Мбит/с \* 86400 с ~ 5,4 ГБ/сут   |
| Отправка медиа-запроса для модели      | 52 \* (0.5 МБ + 75 Б + 226 Б) ~ 28 МБ/c ~ 221 Мбит/с             | 78,2 \* (0.5 МБ + 75 Б + 226 Б) ~ 332 Мбит/с | 221 Мбит/с \* 86400 с ~ 2,4 ТБ/сут   |

## 3. Глобальная балансировка нагрузки

### Обоснования расположения ДЦ

- Сервера OpenAI находятся исключительно на территории США по данным с официального сайта компании [[5](#список-источников)] (Но используется Anycast).
  > ![](img/nslookup.png)
  >
  > ![](img/nsip2.png)
- Пользуясь картой провайдеров [[6](#список-источников)] можно составить список локаций для ДЦ:
  - Сан-Франциско
  - Даллас
  - Нью-Йорк
- Если учитывать, что большая часть трафика идет извне США (см. [Целевая аудитория](#Целевая-аудитория)), то как минимум можно добавить ДЦ для наиболее нагруженных точек мира (Индии, Великобритании), однако в настоящее время ДЦ там нет:
  - Лондон
  - Нью-Дели

Карта с расположением датацентров доступна по [ссылке](https://yandex.ru/maps/?um=constructor%3A53944f7d66f5ff40150fb989d6e66a18acb46857c307120b910fd01c05a8b6a0&source=constructorLink).
![](img/map1.png)
![](img/map2.png)
![](img/map3.png)

### Схема балансировки

Используя GeoDNS для определения региона пользователя (США, Европа или Азия) и BGP Anycast для распределения нагрузки внутри США между тремя датацентрами, можно обеспечить глобальную доступность сервисов и минимизировать задержки за счёт оптимальных маршрутов внутри основного региона.

### Список источников

1. [SimilarWEB](https://pro.similarweb.com/#/digitalsuite/websiteanalysis/overview/website-performance/*/999/1m?webSource=Total&key=chat.openai.com)
2. [ExplodingTopics](https://explodingtopics.com/blog/chatgpt-users)
3. [InvestingInTheWEB](https://investingintheweb.com/education/chatgpt-statistics/)
4. [Language Models are Few-Shot Learners](https://arxiv.org/pdf/2005.14165)
5. [OpenAI](https://platform.openai.com/docs/guides/production-best-practices/improving-latencies#:~:text=Our%20servers%20are%20currently%20located%20in%20the%20US)
6. [IEM](https://www.internetexchangemap.com/)

## 4. Локальная балансировка нагрузки

Для обеспечения отказоустойчивости и балансировки будут использоваться K8s и Nginx.

### Отказоусточивость

- Kubernetes
  - способствует отказоустойчивости, автоматически перезапуская упавшие контейнеры и перераспределяя поды при сбоях узлов, тем самым поддерживая непрерывность работы приложений. Он также обеспечивает балансировку нагрузки и позволяет легко масштабировать приложения, что помогает выдерживать повышенную нагрузку и минимизировать влияние отказов отдельных компонентов.
- Nginx
  - встроенные механизмы отказоустойчивости: Возможность автоматического исключения недоступных бэкендов из пула серверов.

### L7 балансировка

- Nginx
  - поддержка различных алгоритмов балансировки: Nginx позволяет использовать различные методы распределения трафика, такие как Round Robin, Least Connections, IP Hash и другие.
  - терминация SSL, разгрузка бэкенд-серверов: Nginx может выполнять шифрование и расшифровку SSL/TLS-трафика, снимая эту нагрузку с приложений.

### Расчет нагрузки по терминации SSL
