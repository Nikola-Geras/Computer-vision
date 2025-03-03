Проект: "Поиск по изображениям"
Пользователи размещают свои фотографии на хостинге и сопровождают их полным описанием: указывают место съёмок, модель камеры и т. д. Отличительная особенность сервиса — описание: его может предоставить не только тот, кто размещает фотографию, но и другие пользователи портала.

1  Цель:
Разработка поиска референсных фотографий для фотографов. Суть поиска заключается в следующем: пользователь сервиса вводит описание нужной сцены. Сервис выводит несколько фотографий с такой же или похожей сценой.

Для демонстрационной версии нужно выбрать лучшую модель, которая получит векторное представление изображения, векторное представление текста, а на выходе выдаст число от 0 до 1 — и покажет, насколько текст и картинка подходят друг другу.

2  Юридические ограничения
В некоторых странах, где работает компания, действуют ограничения по обработке изображений: поисковым сервисам и сервисам, предоставляющим возможность поиска, запрещено без разрешения родителей или законных представителей предоставлять любую информацию, в том числе, но не исключительно, текстов, изображений, видео и аудио, содержащие описание, изображение или запись голоса детей. Ребёнком считается любой человек, не достигший 16-ти лет.

2.1  Описание данных
Данные лежат в папке /datasets/image_search/.

В файле train_dataset.csv находится информация, необходимая для обучения: имя файла изображения, идентификатор описания и текст описания. Для одной картинки может быть доступно до 5 описаний. Идентификатор описания имеет формат <имя файла изображения>#<порядковый номер описания>.

В папке train_images содержатся изображения для тренировки модели.

В файле CrowdAnnotations.tsv — данные по соответствию изображения и описания, полученные с помощью краудсорсинга. Номера колонок и соответствующий тип данных:

Имя файла изображения.
Идентификатор описания.
Доля людей, подтвердивших, что описание соответствует изображению.
Количество человек, подтвердивших, что описание соответствует изображению.
Количество человек, подтвердивших, что описание не соответствует изображению.
В файле ExpertAnnotations.tsv содержатся данные по соответствию изображения и описания, полученные в результате опроса экспертов. Номера колонок и соответствующий тип данных:

Имя файла изображения.
Идентификатор описания.
3, 4, 5 — оценки трёх экспертов.

Эксперты ставят оценки по шкале от 1 до 4, где 1 — изображение и запрос совершенно не соответствуют друг другу, 2 — запрос содержит элементы описания изображения, но в целом запрос тексту не соответствует, 3 — запрос и текст соответствуют с точностью до некоторых деталей, 4 — запрос и текст соответствуют полностью.

В файле test_queries.csv находится информация, необходимая для тестирования: идентификатор запроса, текст запроса и релевантное изображение. Для одной картинки может быть доступно до 5 описаний. Идентификатор описания имеет формат <имя файла изображения>#<порядковый номер описания>.

В папке test_images содержатся изображения для тестирования модели.

3  План работы
В текущей формулировке задачи вижу два пути решения.

Первый путь. Строим для большую выборку оценок на основе оценок комьюнити и экспертов, далее векторизуем описания и изображения, конкатенируем пары векторов, это становится нашим признаком, ему в соответствие ставится сводная оценка. На этих данных обучаем модель. Что смущает: в дальнейшем, для того, чтобы определить, какое изображение лучше подходит к запросу, через модель придётся прогнать вектора ВСЕХ имеющихся изображений (модель на вход требует описание+изображение)

Второй путь. Строим преобразование вектора описания в вектор изображения (в простейшем случае линейное на линейной регрессии). В этом случае экспертные оценки нам нужны только для того, чтобы отбросить недостоверные пары описание-фото. После обучения модели нам останется только подать на вход вектор запроса и получить вектор изображения. (Интересно было бы попытаться решить обратную задачу и заставить ResNet по имеющемуся вектору восстановить изображение, подозреваю, получится редкостный бред.) Далее наша задача просто найти в базе вектора, максимально близкие полученному. Это и будут искомые изображения. Что смущает: не понятны критерии качества обучения. В процессе обучения мы получим вектор "изображения" и можем сравнить его с вектором правильного изображения, получим между ними некоторое расстояние. Как понять, что расстояние достаточно мало или недопустимо велико? Какие использовать метрики? Пока не понятно.

Выводы

Обе модели показали себя опять очень сходно в смысле прогнозов изображений, но попадания на тестовой выборке не очень хороши.

В целом концепция показала свою жизнеспособность, но она нуждается в существенной доработке. Возможные направления:

Увеличение датасета может существенно улучшить результаты
Использование более глубокой нейросети с более тщательным подбором гирперпараметров может положительно сказаться
Оценки комьюнити не выглядят очень точными и ценными, но они многочислены. Можно подумать, как их заставить работать на повышение точности моделей
