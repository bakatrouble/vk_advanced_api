# VK Advanced API

Интересная штука то получается. Вроде всех этих open-source либ много, но все они
какие-то недоработанные :rage:

Именно так я решил.
Решил и сделал :relaxed:

VK Advanced API предоставляет возможность в полной мере насладиться всеми
фичами API ВК, которое оно само предоставить не может.

Здесь вы увидите:
- Авторизация с помощью Логина и Пароля пользователя, а не только токена
- Улучшение работы некоторых методов и их группирование в один полноценно рабочий метод
- Улучшенный polling эвентов
- Новый тип эвентов - Notifications Events (название авторское)
- Обработку капчи с помощью сервиса RuCaptcha
- Кастомную обработку ошибок
- Качественную и быструю работу
- Гибкую настройку под все нужды

Это и не только вы сможете увидеть тут!

---
### Установка

Очень простая и удобная установка!

**Возможны некоторые ошибки при установке (так уж собрался пакет, что необходимые модули ставяться в конце)**

**Чтобы их избежать, следуйте инструкции ниже**

Выполните следующие команды в вашей консоли
```shell
pip install requests
pip install parsel
pip install captcha_solver
pip install pymitter
pip install lxml
```

И установите сам модуль
```shell
pip install vk_advanced_api
```

Если хотите установить определенную версию, используйте `==version`, где
version - желаемая версия
```shell
pip install vk_advanced_api==1.2.4
```
Чтобы обновить модуль, напишите параметр `--upgrade`, установится последнаяя стабильная версия
```shell
pip install --upgrade vk_advanced_api
```

---
### Авторизация

(Все представленное здесь хорошо описано в директории examples)

Через данные пользователя
```python
# Импорт модуля
import vk_advanced_api

# Экземпляр класса VKAPI
api = vk_advanced_api.VKAPI(
    app_id='your-app-id',
    login='help@ar4ikov.ru',
    password='qwerty',
    captcha_key='your-captcha-key',
    version=5.71,
    warn_level=1
)
```
где:
- app_id - ID Вашего Standalone-приложения ([тут прочтите](http://vk.com/editapp?act=create))
- login - Логин/Email/Телефон юзера
- password - Пароль юзера
- captcha_key - API Ключ к вашему аккаунту на RuCaptcha
- version - Желаемая версия API VK
- warn_level - Уровень лога ошибок, где 1 - вывод в консоль, 2 - вызов ошибок (raise)

При данном типе авторизации будет получен Access Token юзера, так что вы всегда 
сможете получить его
```python
access_token = api.access_token
```

Через токен
```python
# Импортируем модуль
import vk_advanced_api

# Создаем экземпляр класса VKAPI
api = vk_advanced_api.VKAPI(
    access_token='Your-Access-Token',
    captcha_key='your-captcha-key',
    version=5.71,
    warn_level=1
)
```
где:
- access_token - Access Token юзера

---
### Вызов методов

(Все представленное здесь хорошо описано в директории examples)

Всего в модифицированной версии есть пару методов, основную часть которых,
конечно же, составляет само API Вконтактика.

Один из таких - `sendMessage`
(В коде там поищите его описание, мне тут лень описывать это, ну серьезно)
```python

# Тут всё вроде понятно, кроме attachments
# Он описан в examples/upload_media.py

api.sendMessage(
    user_id=1,
    message='Привет, Дуров',
    attachments=[]
    )
```

Чтобы получить класс API, обратимся к ***utils***
```python
>>> utils = api.utils
```
После попытаемся вызвать метод.
Т.к. я постарался поиграться с "магией" Питона ( ***__getattr__*** и ***__call__*** ),
то все методы можно получить просто как методы самой библиотеки. Те, кто знают, что
может ***__getattr__*** и ***__call__*** поймут меня.

```python
>>> utils.messages.send(user_id=1, message='Привет, Дуров!')
```

(а это типо ответ API на ваш запрос)
```shell
364582
```

За подобную идею хочу отблагодарить человека [dimka665](https://github.com/dimka665) и его проект
[vk](https://github.com/dimka665/vk)

---
### LongPolling и обработка эвентов

(Все представленное здесь **полностью** описано в директории examples
Сказал же, что **ПОЛНОСТЬ**!)

Более подробно о технологии `LongPolling VK` читайте [тут](https://vk.com/dev/using_longpoll)

Моя либа предоставляет возможность работать с обработанными эвентами, в частности
направленные на помощь в создании чат-ботов

Например, вот вам эвент ***new_message***

Ключ         | Что означает
-------------|--------------------------------------------------------------
type         | Тип сообщения (public или private)
message_id   | ID сообщения
is_out       | Определяет, явзяется ли сообщение исходящим (True, если да)
args         | Аргументы (просто всё, что разделено пробелом), помогают при создании команд для чат-бота
is_command   | Определяет, является ли сообщение командой (True, если да)
peer_id      | ID диалога (диалогом может быть беседа или личный чат)
from_id      | (только для бесед, вернет None, если личное сообщение) - ID пользователя, который отправил сообщение
body         | Тело эвента в нетронутом виде, которое поступило при запросе на Polling сервер VK
is_acted     | (только для бесед, вернет False, если личное сообщение) - Позволяет определить, является ли этот эвент действием в беседе
attachments  | Вложения


Или вот вам описание ключа ***act*** в эвенте ***new_action***

Ключ         | Что означает
-------------|--------------------------------------------------------------
act          | ID действия
act_mid      | ID юзера, над которым совершили действие
act_from     | ID юзера, который совершил действие
act_text     | Текст, который был передан в действии (обычно новое название беседы)

Также существует особый тип эвентов - `Notifications Events`.

По своей сути данный тип эвентов является тем самым случаем, когда обычный метод становится возможностью получать
все более свежую информацию. Данная функция - `уникальна` и встречается `только` в данной библиотеке.
(За идею спасибо [ему](https://vk.com/enotakin))

**Примечание**

***Данный функционал находится в тестировании и возможно нуждается в правках.***

Ниже представлено описание ключей данного эвента -> ***new_notification***

Ключ         | Что означает
-------------|--------------------------------------------------------------
user_id | ID пользователя, который совершил действие. Параметр `user_ids` не передается, если был передан текущий
user_ids | Список, содержащий ID пользователей, совершивших данное действие в один промежуток времени или там, где это необходимо. Параметр `user_id` в данном случае не передается
type | Тип действия, подробнее на https://vk.com/dev/notifications.get
date | Дата события по UNIX
body | Тело события (Для разных типов событий содержатся разные поля)
parent_id | ID материала, к которому появился ответ
parent | Информация о материале, к которому появился ответ

(На самом деле я простот игрался с таблицами, красиво выглядят...)

---
### Заключение

Не поймите меня неправильно, но я просто уже устал от того, что для многих понимания
`простота` и `удобство` утратило свой смысл. Большинство существующих либ для работы
с API Вконтакта либо слишком простые и засчёт этого мало чем отличаются от сырых
запросов, либо настолько накрученные, что рядовой юзер может потеряться во всем разнообразии вещей.

Я не поливаю их говном, это моё личное мнение.
НО именно это мнение подтолкнуло сделать меня нечто подобное. :computer:

Естественно, либа будет обновляться, улучшаться.
На этом пока всё.
