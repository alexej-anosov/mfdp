# Speech-to-text сервиса для транскрибирования корпоративных звонков в IT компаниях

*Из предложенных тем, я выбрал четвертую, связанную с обработкой звука. Изначально в ней предлогалось сделать сервис, удаляющий шум из аудиозаписи. Однако, в ходе работы, мне захотелось измененить тему на разработку сервиса, по распознаванию речи. Возможность такого изменения была согласована с организаторами курса.*

## Структура репозитория
Репозиторий на данный момент состоит из двух папок:

1. service. в ней находится сам сервис
2. notebooks. в ней находятся все файлы и заметки отражающие ход работы.

Инструкция по запуску находится в папке service.




## Описание идеи
За последние три года видеозвонки прочно вошли в нашу жизнь и в будущем, как мне кажется, будут составлять всё большую долю нашей профессиональной коммуникации. Видеозвонки имеют ряд преимуществ над обычными встречами, однако, можно постараться сделать их еще более удобными и эффективными. Среди возможных улучшений одним из самых очевидных является предоставление пользователям текстовых версий прошедших встреч. Такой функционал позволит:
1. сконцентрироваться во время звонка на обсуждении важных тем, не отвлекаясь на ведение заметок/конспектов
2. сотрудникам, которые не были на встрече, быстро узнать, что на ней обсуждалось
3. сотрудникам, которые опоздали, быстро ознакомиться с тем, что было до их прихода, и включиться в обсуждение (если транскриирование происходит синхронно)
4. хранить важные встречи гораздо компактнее - в виде текста (если визуальная составляющая не играет важной роли).
Также имея хорошие текстовые расшировки, можно предоставлять и другие сервисы. Например, суммаризацию встреч, или автоматизацию некоторых действий (например, автоматически создавать таски после встречи).

В своем проекте, я попытался найти подходы к созданию сервиса распознавания речи для корпоративных звонков в IT компаниях.
Речь в данной сфере наполнена большим количеством специальной лексики: названия программ, фреймворков, моделей, и т.д. Это усложняет возможность создания расшифровок хорошего качества. Универсальные модели плохо справляются с этой задачей, так как в стандартных датасетах нет соответствующей лексики. Также, часть таких слов должна транскрибироваться с помощью латинских букв. Далеко не все решения имеют возможность совмещать латиницу и кириллицу.

С точки зрения бизнес составляющей, создание такого сервиса кажется оправданным, так как часть популярных сервисов (Zoom, Google Meet) являются иностранными. Для иностранных сервисов работа с кириллицей всегда представляла сложности (Google проиграл Яндексу российский рынок). Тем более подобные сервисы не будут много заниматься кириллическими моделями в условиях нынешних санкций, когда российский рынок явно не является приоритетным. Таким образом, пространство для сторонних сервисов будет открытым еще долгое время. 

Также стоит отметить, что идеи, реализованные в ходе данного проекта, могут быть в будущем использованы для дообучения моделей распознавания речи для других сфер, в которых присутствует большое количество специальной лексики.

## Техническая часть
### ML-часть
Пайплайн состоит из двух моделей:
1. Модель диаризации (разделения на спикеров)
2. Модель распознавания речи
Модель диаризации выдает временные метки разделения на спикеров. Согласно этим меткам исходное аудио нарезается на сегменты. В одном сегменте - одна реплика одного спикера. Далее эти сегменты передаются в модель распознавания. Результаты склеиваются и форматируются согласно заданным правилам, и выдаются пользователю.

##### Модель диаризации
В качестве модели диаризации была выбрана open-source модель фреймворка Pyannote. Она признается сообществом одной из лучших (на основе изучения обсуждений в [ТГ-чатике "Распознавание и синтез речи"](https://t.me/speech_recognition_ru)). Модель в нынешней работе не модифицировалась. Однако, это может стать одним из направлений дальнейшего улучшения сервиса.

##### Модель распознавания речи
На основе [сравнения](https://alphacephei.com/nsh/2023/01/22/russian-models.html) открытых моделей для распознавания речи на различных датасетах было выделено три наиболее интересных варианта:
1. [Vosk](https://alphacephei.com/vosk/models)
2. [NVIDIA Conformer-Transducer Large (RU)](https://huggingface.co/nvidia/stt_ru_conformer_transducer_large)
3. [Whisper](https://openai.com/research/whisper)

При дальнейшем изучении выбор пал на модель Whisper от OpenAI, так как: 
1. она легко обучается(в отличие от Vosk, основанной на системе инструментов Kaldi) (лектор [курса](https://www.youtube.com/playlist?list=PLEwK9wdS5g0oE9htlwY-WarI5_jeH818j) по обработки аудио от ВШЭ ФКН, упоминает, что "кто с Kaldi работал, тот в цирке не смеется". В условиях ограниченного времени, я решил поверить ему на слово и не проверять на себе.)
2. умеет из коробки включать в распознавание русского языка латинские символы, что позволяет ее дообучать, а не переучивать с нуля (в отличие от модели от Nvidia). 
Кажется, при создании MVP, лучше не искать дополнительных сложностей, особенно, если они не сулят значительного улучшения качества.

На данный момент использовалась модель Whisper размера Small (третья по величине из пяти), так как обучение старших моделей не поместилось 15gb GPU, которые доступны бесплатно на платформах Kaggle или Colab. Если будут хорошие результаты при обучении модели small, можно будет арендовать на несколько дней сервер и поэкспериментировать с большой моделью, используя обкатанную технологию.


#### Данные
Часть данных для обучения была взята из открытых источников.
1. Golos (crowd) от Сбера (часть)
2. Sova dataset (audiobooks) от sova.ai
3. CommonVoice от Mozilla
(суммарно около 1200 часов)

Согласно оригинальной [статье Whisper](https://cdn.openai.com/papers/whisper.pdf) из данных датасетов при исходном обучении использовался только CommonVoice, поэтому я решил, что показать их модели будет не лишним, чтобы улучшиить качество распознавания для русского языка в целом.

Однако, главная задача состояла в том, что найти откуда взять данные из целевой сферы.

#### Youtube
Одной из идей было использование данных с платформы Youtube. Можно взять видео из нужной сферы (митапы, мок-интервью, интервью с гостями из данной сферы). Автоматические субтитры есть почти на всех видео, и кажется, они неплохого качества. Даже часть специальной лексики правильно распознают (не всю, конечно, но лучше, чем ничего). 

Есть программы, которые позволяют скачивать необходимые файлы (аудио и субтитры) с Ютуба целыми плейлистами или даже каналами, поэтому, имея автоматический обработчик, можно собрать сколько угодно данных. (Для обучения Whisper, кстати, использовалось 680 тысяч часов данных)). 
Мной был написан такой обработчик, который из папки, содержащей файлы субтитров и аудио-файлы, создает датасет на платформе HuggingFace. Для первых экспериментов собрано 150 часов данных.

1 час был выделен в тестовую выборку и размечен вручную. Сравнение ручной разметки с автоматическими субтитрами показало ошибку wer=0.145. Это несколько меньше, чем даже ошибка самой большой модели Whisper(wer=0.187), не говоря уже о модели Small, которую я собирался обучать. В связи с этим я решил оставить данные с Youtube в обучающей выборке.

##### Обучение на открытых данных + данных с Youtube

###### Аугментации обучающей выборки
К данным во время обучения применялись следующие аугментации:
* замедление/ускорение (0.8x - 1.25x)
* наложение гаусового шума (разновидность белого шума)
* уменьшение/увеличение громкости
* Pitch Shifting (изменение тембра)
* ApplyImpulseResponse (симуляция акустики различных помещений) Эффекты взяты из набора [MIT](http://mcdermottlab.mit.edu/Reverb/IR_Survey.html)
* BandPassFilter (фильтр частот)
* ClippingDistortion (Дисто́ршн — звуковой эффект, достигаемый искажением сигнала путём его «жёсткого» ограничения по амплитуде, или устройство, обеспечивающее такой эффект.)
* наложение шумовых звуков (звуки окружающей среды из (датасета)['https://github.com/karolpiczak/ESC-50'](пылесос, шум машин, плач ребенка, и тд, всего 50 категорий))

###### Аугментации тестовых выборок
Для создания аугментированных тестовых выборок применялся более ограниченный набор аугментаций:
* наложение гаусового шума 
* ApplyImpulseResponse 
* наложение шумовых звуков (была выделена тестовая выборка шумов из исходного датасета, чтобы не было прямого переподгона)

###### Список тестовых датасетов
Чтобы наиболее объективно оценить результаты экспериментов, делались оценки на 5 разных датасетах. 4 из них были test-сплитами обучающий датасетов и 1 датасет (записи звонков) был добавлен для еще большей объективности. Ничего похожего на данный датасет в обучающей выборке не было. 

Для каждого датасета использовать как изначальная версия, так и аугментированная.

###### Результаты
При обучении использовались не все 1370 часов, а порядка 250. Кажется, для файн-тюнинга этого вполне достаточно.

С результатами вы можете ознакомиться по [ссылке](https://docs.google.com/spreadsheets/d/1qVgNXQ9315JK8VqBGJyCbRVeZYzFEwyU4HxO59FsNjg/edit#gid=0)

В целом, качество распознавания улучшилось. Однако, именно у целевого датасета Youtube качество немного ухудшилось. Более того, качество ухудшилось именно за счет ухудшения распознавания специальной лексики. Видимо от такого большого количества русскоязычного материала, модель стала забывать английский, поэтому термины стали расшифровываться кириллицей.

Возможно все-таки качество субтитров Ютуба является не удовлетворительным. Той части правильно распознанных терминов не хватает для хорошего обучения. Доразмечать автоматические субтитры, мне кажется, не рациональным. По моему опыту исправление ошибок занимает в несколько раз больше времени, чем количество материала, которое надо разметить. Учитывая, что такую разметку, надо еще и перепроверять, это слишком дорого.  Но есть и другой способ...

#### Текстовые данные
Кажется, что дешевле будет попросить людей на Толоке наговорить нужный текст, а не доразметить существующее аудио. Но откуда взять столько текстовых данных?

Я решил спарсить Хабр. Всего для эксперимента спарсил около 5000 статей. На данный момент сфокусировался на написанных латиницей, хотя и среди кириллических слов наверяка есть редкие, которые было бы полезно разметить. Далее я
1. посчитал число вхождений каждого англоязычного слова. 
2. Удалил слова, которые входят в 333 тысячи самых распространенных слов английскоязычного интернета (ушли такие слова как Microsoft, Android, IOS, и тд) (кажется, их распознать можно и без допразметки.)
3. удалил слова, встретившиеся меньше 3 раз и в которых меньше 5 букв. 
Субъективно, получился достаточно хороший словарь. 
4. после этого выделил предложения, содержащие такие слова, для последующей разметки голосом.

Разметить на Толоке достаточно предложений для обучающей выборки мне явно не позволяет бюджет. Однако, в планах было разметить хотя бы часть для тестовой выборки. К сожалению, столкнулся с тем, что Толока просит публиковать соглашение об обработке персональных данных, так как голос человека является частью таких данных. Времени до сдачи проекта оставалось мало - разобраться что это за соглашение у меня не получилось. Поддержка Толоки предоставить шаблон не смогла.

Однако есть еще один вариант - сделать озвучку с помощью моделей text-to-speech. Это нормальный метод, современные TTS модели обладают достаточным качеством, чтобы на них можно было обучать STT. Об этом говорится, например вот в [этой](https://arxiv.org/abs/1811.02050) статье от Google Research. 

Попробовав генерировать озвучку с помощью TTS [модели от Facebook](https://huggingface.co/facebook/tts_transformer-ru-cv7_css10) я обнаружил, что английски слова в русском тексте модель читает с явным английским акцентом, что может мешать способности модели узнавать эти слова при другом произношении. Также, он может не знать как принято произносить слова, и произносить их согласно правилам английского языка. Например 'nosql' модель произносит как 'носкл' вместо ожидаемоего ~'ноу-эс-ку-эл'.

Решить эту проблему, кажется, можно только создав словарь, в котором собранные англоязычные термины имели транскрипции из кириллицы. Кажется абсолютного число терминов покрывается словарем в несколько тысяч слов. Разметить такое количество не так долго и дорого. Имея же такой словарь можно будет:
1. генерировать тексты гораздо лучшего качества с помощью TTS моделей
2. снизить стоимость при разметке на Толоке и повысить ее качество, так как толокеры будут точно читать правильно
3. использовать модели только с кириллическим токенизатором, без обучения с нуля
4. такие данные не будут входить в конфликт с датасетом Голос при обучении, в котором все англоязычные слова приведены к кириллице

Проблема будет заключаться только в том, что ошибка модели на один символ уже не позволит сметчить кириллическое предсказание с исходным словом на английском, которое мы хотим показывать в итоговой расшифровке. Но, наверное, эту проблемы можно постараться решить либо эвристиками, либо дополнительной маленькой языковой моделькой. Но это уже совсем другая история, которой, я надеюсь, я займусь во время учебы в магистратуре ИТМО :)

#### Результаты работы над ML-частью
В ходе работы 
1. был написан распарщиватель платформы Youtube. 
2. проверена гипотеза о пользе таких данных в обучающей выборке.
3. сформирована следующая гипотеза о возможности использования данных, сгенерированных STT моделью
проведена часть подготовительной работы для ее проверки. 
К сожалению, довести вторую гипотезу до каких-то результатов не хватило времени. Но ее хорошая реализация, как мне кажется, требует значительно больше времени, чем было на прохождение курса. Таким образом, я считаю работу, проделанную с ML-частью, неплохим фундаментом для своих будущих экспериментов.

### Сервис
Бэкенд реализован с помощью фреймворка FastAPI.

Фронтенд реализован с помощью инструментов Bootstrap, а также языка программирования JavaScript.

На данный момент можно загрузить файл с компьютера или записать с помощью микрофона.

Распознанный текст выводится на экран. Его можно подкорректировать вручную и сохранить в виде txt-файла.

Также для сервиса настроены первые версии
1. мониторинга с помощью Prometheus
2. логирования с Loki
3. визуализации метрик и логов с помощью Grafana
4. нагрузочное тестирование с помощью Locust

Результаты нагрузочного тестирования на данный момент не имеют большого смысла, так как тестирование проводилось на сервере без GPU и с не очень сильным процессором. Однако, наличие настроенного тестирования позволит легко его выполнить, когда будет такая потребность.

В будущем хотелось бы добавить версию приложения в виде расширения для браузера, которое могло бы вести полноценную запись конференций. Сейчас доступна только запись микрофона.