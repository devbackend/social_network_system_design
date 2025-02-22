# System Design социальной сети для курса по System Design

## Functional Requirements

- пользователь сможет публиковать посты
- пост состоит из фото, текста и геолокации
- пользователи смогут просматривать, оценивать и комментировать посты других пользователей
- можно будет подписаться/описаться на других пользователей
- пользователи смогут искать посты по геолокации
- пользователи смогут просматривать ленту постов из своих подписок

## Non-functional Requirements

- 10 000 000 DAU через год и продолжит расти
- Запуск только в странах СНГ
- Доступность 99.95% времени - около 4х часов downtime в год
- Храним посты постоянно
- Публикация поста, комментария, реакции и подписка должны работать за 1 секунду
- Новый пост должен появляться в ленте подписчиков через 30 секунд
- Запрос своей ленты должен работать за 2 секунды
- Запрос ленты по геолокации должен работать за 3 секунды
- Пользователи будут в среднем
    - публиковать 1 пост в день
        - в среднем к посту будет прикреплено 5 фото
        - в среднем размер поста будет 500 символов
    - просматривать свою ленту 10 раз в день
    - просматривать ленту по геолокации 2 раза в день
    - отправлять 15 оценок в день постам
    - писать 5 комментариев в день
    - подписываться/отписываться 1 раз в день
- будет сезонность - летний период и рождественская неделя
    - в этот период ожидаем в среднем
        - 2 поста в день
        - 8 фото к посту
        - 1200 символов размер поста
        - просматривать ленту 15 раз в день
        - совершать 5 запросов ленты по геолокации в день
        - ставить 25 оценок постам в день
        - писать 10 комментариев
        - подписываться/отписываться 3 раза в день

## Лимиты

- размер поста - 1500 символов
- размер комментария - 250 символов
- 10 фото к посту
- размер фото - 500kB
- количество постов при запросе пагинации в ленте - 20

## Описание хранения в БД

Post (1530 byte)

- id - 8 byte
- user_id - 8 byte
- text - 1500 byte
- location_id - 8byte

Photo (30 byte)

- id - 8 byte
- post_id - 8 byte
- uri - 10 byte

Comment (280 byte)

- id - 8byte
- user_id - 8 byte
- post_id - 8byte
- text - 250 byte

Like (20 byte)

- post_id - 8byte
- user_id - 8 byte

## Расчет нагрузки через год

Для упрощения расчетов

- считаем что в сутках 100 000 секунд (вместо 86 400)

| Тип взаимодействия            | Обычное время                                                | Сезонные повышения нагрузки                                  |
|-------------------------------|--------------------------------------------------------------|--------------------------------------------------------------|
| Публикация поста              | 10 000 000 * 1 / 100 000 = 100 RPS                           | 10 000 000 * 2 / 100 000 = 200 RPS                           |
| Сохранение файлов             | 10 000 000 * 5 / 100 000 = 50 000 000 / 100 000 = 500 RPS    | 10 000 000 * 8 / 100 000 = 80 000 000 / 100 000 = 800 RPS    |
| Просмотр ленты геолокации     | 10 000 000 * 2 / 100 000 = 20 000 000 / 100 000 = 200 RPS    | 10 000 000 * 5 / 100 000 = 50 000 000 / 100 000 = 500 RPS    |
| Просмотр своей ленты подписок | 10 000 000 * 10 / 100 000 = 100 000 000 / 100 000 = 1000 RPS | 10 000 000 * 15 / 100 000 = 150 000 000 / 100 000 = 1500 RPS |
| Отправка оценок               | 10 000 000 * 15 / 100 000 = 150 000 000 / 100 000 = 1500 RPS | 10 000 000 * 25 / 100 000 = 250 000 000 / 100 000 = 2500 RPS |
| Отправка комментариев         | 10 000 000 * 5 / 100 000 = 50 000 000 / 100 000 = 500 RPS    | 10 000 000 * 10 / 100 000 = 100 000 000 / 100 000 = 1000 RPS |
| Подписка/отписка              | 10 000 000 * 1 / 100 000 = 100 RPS                           | 10 000 000 * 3 / 100 000 = 30 000 000 / 100 000 = 300 RPS    |

## Расчет трафика через год

Для упрощения расчетов

- все переводы байтов округляем до 1000 (вместо 1024)

| Тип взаимодействия                    | Обычное время                                              | Сезонные повышения нагрузки                          |
|---------------------------------------|------------------------------------------------------------|------------------------------------------------------|
| Публикация поста                      | 100 * 1530 bytes = 153000 bytes = 153 kb/s                 | 200 * 1530 = 306000 bytes = 306 kB/s                 |
| Сохранение файлов                     | 500 * 500 000 bytes = 250 000 000 bytes = 250 mb/s         | 800 * 500 000 byte = 400 000 000 bytes = 400 mb/s    |
| Просмотр ленты геолокации (посты)     | 200 * 20 * 1530 bytes = 6 120 000 bytes = 6.2 mb/s         | 500 * 20 * 1530 bytes = 15 300 000 = 15.5 mb/s       |
| Просмотр ленты геолокации (фото)      | 200 * 20 * 500 000 bytes = 2 000 000 000 bytes = 2 gb/s    | 500 * 20 * 500 000 bytes = 5 000 000 000 = 5 gb/s    |
| Просмотр своей ленты подписок (посты) | 1000 * 20 * 1530 bytes = 30 600 000 bytes = 31 mb/s        | 1500 * 20 * 1530 bytes = 45 900 000 bytes = 46 mb/s  |
| Просмотр своей ленты подписок (фото)  | 1000 * 20 * 500 000 bytes = 10 000 000 000 bytes = 10 gb/s | 1500 * 20 * 500 000 bytes = 15 000 000 000 = 15 gb/s |
| Отправка оценок                       | 1500 * 20 byte = 30 000 bytes = 30 kb/s                    | 2500 * 20 byte = 50 000 bytes = 50 kb/s              |
| Отправка комментариев                 | 500 * 280 = 140 000 bytes = 140 kb/s                       | 1000 * 280 = 280 000 bytes = 280 kb/s                |
| Подписка/отписка                      | 100 * 10 = 1 000 bytes = 1 kb/s                            | 300 * 10 = 3 000 bytes = 3 kb/s                      |

## Расчет объема хранилищ через год

Считаем по повышенной сезонной нагрузке

Для упрощения расчетов

- считаем что в сутках 100 000 секунд (вместо 86 400)
- оставляем запас x1.5 - считаем для 500 дней (вместо 365)
- в году - 500 * 100 000 = 50 000 000 секунд
- все переводы байтов округляем до 1000 (вместо 1024)

| Тип данных  | Расчет                                                          | Объем   |
|-------------|-----------------------------------------------------------------|---------|
| Посты       | 306 000 bytes/s * 50 000 000 = 15 300 000 000 000 bytes         | 15.5 Tb |
| Файлы       | 400 000 000 bytes/s * 50 000 000 = 20 000 000 000 000 000 bytes | 20 Pb   |
| Оценки      | 50 000 bytes/s * 50 000 000 = 2 500 000 000 000 bytes           | 2.5 Tb  |
| Комментарии | 280 000 bytes/s * 50 000 000 = 14 000 000 000 000 bytes         | 14 Tb   |
| Подписки    | 3 000 bytes/s * 50 000 000 = 150 000 000 000 bytes              | 150 Gb  |
