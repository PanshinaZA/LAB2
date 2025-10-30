# Лабораторная работа №2. Проектирование и реализация клиент-серверной системы. HTTP, веб-серверы и RESTful веб-сервисы
## Цель работы: 
изучить методы отправки и анализа HTTP-запросов с использованием инструментов telnet и curl, освоить базовую настройку и анализ работы HTTP-сервера nginx в качестве веб-сервера и обратного прокси, а также изучить применить на практике концепции архитектурного стиля REST для создания веб-сервисов (API) на языке Python.
## Оборудование и программное обеспечение:
1. Операционная система: Ubuntu 20.04.6 LTS (в рамках предоставленного образа).
3. Сетевые утилиты: telnet, curl.
4. Веб-сервер: nginx.
5. Среда разработки:
- Интерпретатор Python 3.8+.
- Система управления пакетами python3-pip.
- Инструмент для создания виртуальных окружений python3-venv.
- Микрофреймворк Flask для реализации REST API.
6. Доступ к сети Интернет.

## Вариант 17. Задача:
1. HTTP-анализ. Анализ ответа API randomuser.me/api для получения случайного пользователя.
2. Разработка REST API. API для "Расписание занятий" (сущность: id, subject, teacher, time).
3. Настройка Nginx. Настроить Nginx как обратный прокси для Flask API.
## Ход работы
1. Первоначально создадим директорию проекта и перейдем в нее, а также установим утилиту для работы с веб-ресурсами через сетевые протоколы:

<img width="787" height="500" alt="image" src="https://github.com/user-attachments/assets/261bb0a6-851a-41c1-bc5b-2c7fdf809ae1" />

2. Далее проанализируем JSON-ответ от API randomuser.me/api для получения случайного пользователя при помощи команды:
```
curl https://randomuser.me/api/
```
Она выведет информацию об...

<img width="1176" height="182" alt="image" src="https://github.com/user-attachments/assets/ecd14449-2530-40bb-ba97-e290d134ef9c" />

Затем введем команду:
```
curl -v https://randomuser.me/api/
```
Она выведет подробную информацию о HTTP-запросе и ответе:

<img width="701" height="504" alt="image" src="https://github.com/user-attachments/assets/43fd581b-866c-4e0b-8862-e0946bdab3a5" />
<img width="1158" height="525" alt="image" src="https://github.com/user-attachments/assets/9c70536c-e87a-4824-96c3-abcd91a3684c" />
<img width="1155" height="48" alt="image" src="https://github.com/user-attachments/assets/77e703ef-b052-4890-ae1d-87d0456cd423" />

А также выведем запросом информация в формате json:
```
curl -s https://randomuser.me/api/ | jq
```

<img width="829" height="439" alt="image" src="https://github.com/user-attachments/assets/4d250b69-f203-4708-8126-b986e7dab3cd" />
<img width="817" height="440" alt="image" src="https://github.com/user-attachments/assets/801436e4-665a-459d-a1c2-990d9922de69" />
<img width="704" height="255" alt="image" src="https://github.com/user-attachments/assets/ba64cd7d-1e18-4a60-bd2e-a9931c081fc2" />

Заметим, что запросы действительно показывают каждый раз случайного пользователя:

<img width="782" height="458" alt="image" src="https://github.com/user-attachments/assets/01917590-75bf-4da8-948c-80963c9b2988" />

Следующий запрос покажет все заголовки и код ответа:
```
curl -i https://randomuser.me/api/ | jq
```
<img width="1184" height="411" alt="image" src="https://github.com/user-attachments/assets/fd7a8db4-89b6-44c9-b029-eb1253a1145d" />

3. Далее произведем установку и базовую настройку Nginx и убедимся, что статус active (running), с помощью команды:
```
sudo systemctl status nginx
```

<img width="938" height="458" alt="image" src="https://github.com/user-attachments/assets/5a9bf4b8-22d2-4c7c-941e-01ad87ff359c" />

4. Убедимся, что при открытии браузере http://localhost, выведена приветственная страница Nginx:

<img width="1358" height="522" alt="image" src="https://github.com/user-attachments/assets/6f5e5191-6089-4448-8ac2-c192126b3606" />

5. Для реализации простого REST API на Python с использованием Flask произведем установку окружения, а также его активацию при помощи команды:
```
source venv/bin/activate
```
<img width="853" height="228" alt="image" src="https://github.com/user-attachments/assets/94a7bb14-d1f9-4974-9724-73d3e43b3f55" />

6. Произведем установку Flask:
<img width="1215" height="380" alt="image" src="https://github.com/user-attachments/assets/6ff0a190-37af-4f36-897f-663a5400c46e" />

7. Создаем файл app.py с помощью команды nanno app.py  и прописываем в нем следующий код:
```
from flask import Flask, jsonify, request
from datetime import datetime

app = Flask(__name__)

# В памяти храним расписание
lessons = [
    {"id": 1, "subject": "Математика", "teacher": "Иванов И.И.", "time": "2025-11-05T10:00:00"},
    {"id": 2, "subject": "Физика", "teacher": "Петрова А.С.", "time": "2025-11-05T12:00:00"}
]
next_id = 3

@app.route('/api/lessons', methods=['GET'])
def get_lessons():
    """Получить всё расписание"""
    return jsonify({"lessons": lessons})

@app.route('/api/lessons', methods=['POST'])
def create_lesson():
    """Добавить новое занятие"""
    global next_id
    data = request.get_json()

    if not data:
        return jsonify({"error": "Тело запроса должно быть в формате JSON"}), 400
    if not all(k in data for k in ("subject", "teacher", "time")):
        return jsonify({"error": "Требуются поля: subject, teacher, time"}), 400

    try:
        # Проверка корректности формата времени
        datetime.fromisoformat(data["time"].replace("Z", "+00:00"))
    except ValueError:
        return jsonify({"error": "Поле 'time' должно быть в формате ISO 8601"}), 400

    new_lesson = {
        "id": next_id,
        "subject": data["subject"],
        "teacher": data["teacher"],
        "time": data["time"]
    }
    lessons.append(new_lesson)
    next_id += 1
    return jsonify(new_lesson), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```
<img width="1196" height="466" alt="image" src="https://github.com/user-attachments/assets/6c7a274e-c663-4907-8194-76655067faea" />

8. Запустим Flask приложение при помощи команды:
```
python3 app.py
```

<img width="986" height="215" alt="image" src="https://github.com/user-attachments/assets/462d59cd-c833-49ac-b2d5-101c189f74d2" />

9. Проверим работоспособность приложения путем вывода расписания занятий с помощью команды:
```
curl -s http://127.0.0.1:5000/api/lessons | jq
```

<img width="1201" height="476" alt="image" src="https://github.com/user-attachments/assets/194d624e-8635-46d3-9fef-2fee5f189200" />

10. Теперь настроим Nginx так, чтобы все запросы, приходящие на http://localhost/api/, перенаправлялись на Flask-приложение. Откроем конфигурационный файл при помощи команды:
```
sudo nano /etc/nginx/sites-available/default
```
Добавим блок:
```
location /api/ {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

<img width="1086" height="175" alt="image" src="https://github.com/user-attachments/assets/3268c2a4-35d1-479f-9fe0-3515ec879e7d" />

11. Проверим синтаксис конфигурации и перезапустим Nginx:

<img width="563" height="74" alt="image" src="https://github.com/user-attachments/assets/728bd6ab-aed4-4141-988a-0ef2936ebf92" />

12. Откроем новый терминал (не закрывая тот, где работает Flask). Теперь все запросы мы будем делать к Nginx (порт 80), а не напрямую к Flask (порт 5000). Сделаем запрос для получения списка расписания занятий:
```
curl http://localhost/api/lessons | jq
```
<img width="651" height="337" alt="image" src="https://github.com/user-attachments/assets/4073cfcb-0d82-4a46-be1c-0596de574802" />

А также создадим новое занятие:
```
curl -X POST -H "Content-Type: application/json" \ -d '{"subject": "Английский", "teacher": "Кузнецова Е.Н.", "time": "2025-11-07T09:00:00"}' \ http://localhost/api/lessons | jq
```
<img width="658" height="211" alt="image" src="https://github.com/user-attachments/assets/b106d298-7a1d-4b19-9801-30e24e87f695" />

13. Обратим внимание на логи в терминале Flask:

<img width="578" height="318" alt="image" src="https://github.com/user-attachments/assets/3310f7e6-b0a4-4ef8-a6e9-9c9c618d946d" />

## Выводы
В ходе выполнения лабораторной работы были изучены методы отправки и анализа HTTP-запросов с использованием инструментов telnet и curl, освоены базовая настройка и анализ работы HTTP-сервера nginx в качестве веб-сервера и обратного прокси.
