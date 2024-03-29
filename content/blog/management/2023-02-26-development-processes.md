---
layout: post
title: Любые процессы разработки должны быть явными
category: management
tags: [teamlead,opinion,development,processes]
date: "2023-02-26"
---

![Типичный режим работы без очевидных процессов](/images/blog/management/2023-02-26-processes/main.jpg)

_Картинка взята [отсюда](https://t.me/komikaki)_

> Очевидные вещи нужно проговаривать

Так говорил мой тимлид, закончивший юрфак, но ушедший в айти. Говорил он так о процессах разработки: свод правил, по которым работает команда. Этакий кодекс программиста отдела N. Этот свод правил должен быть публичным и каждый должен знать, где его прочесть. Но зачем нужно описывать то, что и так всем известно? Давайте обсудим.

Правила работы в команде есть всегда, даже если они нигде не описаны и никем не проговорены. Так складывается исторически, что Ваня лучше знает платежи, Петя - как настроить тестовое окружение, а Юля - что делать, если нашел баг. И тот, кто уходит в отпуск, начинает получать сообщения в мессенджеры: "сорри что пишу, но .....". В такой среде легко допустить ошибку, особенно новичкам. Еще хуже, когда даже не знаешь, кого можно спросить в критичный момент.

## О чем писать?

Чтобы одни сотрудники спокойно отдыхали в отпуске, а остальные знали, что делать в той или иной ситуации, команды описывают все процессы разработки, которые применяют. О чем пишут в кодексе:

- Какая брэнч-стратегия применяется в команде,
- Какие принципы работы декларируем,
- Как работаем над ошибками, когда их находим или когда совершаем,
- Пишем ли тесты, обязательны ли они,
- Какие ритуалы есть в команде, для чего они были придуманы.

Если не сделать подобного "очевидного" описания того, что и так уже есть, то можно столкнуться с тем, что каждый видит процессы по-своему. А раз видение разное, то и поведение участников команды будет разное в критических ситуациях. Для того и нужно описать видение процессов. К тому же, это хороший повод пересмотреть то, что уже сложилоь исторически - а вдруг процесс устарел или есть лучше механики? Когда процессы описаны, каждый знает, что делать в разных ситуациях. Совершил ошибку, хотя следовал процедурам команды? Виноваты процедуры, их нужно доработать. На код-ревью зацепился с коллегой насчет архитектуры? Обращаешься к техлиду в соответствии с процедурами, и тот экономит ваше время и нервы. Описание процессов нужно спецам любого уровня и любой роли:

- джунам кодекс нужен, потому что меньше возникнет вопросов и меньше способов облажаться,
- миддлам кодекс нужен, чтобы джуны реже задавали одни и те же вопросы,
- сеньорам кодекс нужен, чтобы никто не отвлекал во время отпуска по рабочим вопросам.

Написать кодекс будет несложно, если начать с малого. Можно и нужно описывать процессы постепенно по мере их декларирования или пересмотра. Начать можно с общих принципов, а затем дописывать в случае, если произошел случай, не описанный в кодексе. Для примера предлагаю посмотреть тот, который [написал](https://github.com/maximgorbatyuk/project-documents/blob/main/code-of-conduct.md) когда-то я для своей команды. Далее я покажу некоторые ключевые принципы работы и объясню, почему они - ключевые.

## Кодекс команды

> 5 принципов работы
>
> - Принимаем реальность и работаем с ней.
>
> - Нет ничего страшного в том, что мы допускаем ошибки; страшно, когда мы не учимся на них.
>
> - Мы выявляем проблемы и не миримся с ними. Мы исправляем их и делаем выводы.
>
> - Мы используем инструменты и утвержденные процедуры, чтобы регламентировать выполнение работы.
>
> - Явное лучше неявного

В начале я декларирую общие принципы работы. Они могут показаться очевидными, однако лучше их проговорить вслух. Тогда каждый будет уверен, что коллеги тоже в курсе о принципах работы в команде.

> 1.1. Мы следуем принятым принципам, независимо от согласия с ними
>
> Персональное несогласие — не причина отступать от принципов. Тем не менее, можно начать обсуждение любого принципа, если изменились обстоятельства или принцип перестал быть полезным. __Выполняй или объясняй__ - мы можем отступать от установленных правил или процедур, только если мы можем дать объективную причину своим действиям.

В первом же пункте я пишу о том, что каждый, продолжая работать в команде, соглашается с правилами или предлагает их исправить и/или улучшить. Иначе - лучше отказаться от работы в команде. Некое "пользовательское соглашение", где, продолжая работать с программой, ты соглашаешься с условиями ее использования.

> 1.2. Мы открыто говорим о проблемах и рисках
>
> Не согласен с техническим решением или план невозможно выполнить — скажи. Сломал базу или профакапил сроки — скажи сразу. Тимлид пришел с дурацким предложением — не молчи. Часто кажется, что проще согласиться или промолчать. Помни, что это приведёт к ещё большим проблемам позже.

> 1.13. Ошибиться - нестрашно. Страшно повторить ошибку
>
> Мы люди и мы ошибаемся. Мы ошибаемся вследствие незнания, невнимательности или плохого настроения. Любая ошибка простительна, если она совершается впервые. Главное - разобрать причины ошибки и сделать из них вывод.

Этот пункт важный, потому что люди часто боятся сообщать о своих ошибках до последнего. Атмосфера в команде должна быть такая, что страх этот будет безосновательный. Этими пунктами кодекса я как раз подчеркиваю это.

> 1.16. Увидел баг - оформи багрепорт
>
> Не оставляй "как есть" и не разбирайся с причинами - это занимает время. Оформи багрепорт с шагами для воспроизведения и описанием ожидаемого результата и того результата, который проявляется в системе.

В любой команде, где я работаю, я побуждаю тиммейтов оформлять багрепорты или оставлять тудушки в коде. Даже если багрепорт уже был заведен, то лучше закрыть дубликат, чем не сообщить о баге совсем. О том, как оформлять багрепорты, я писал [тут](/blog/development/2021-12-28-how-to-create-effective-bug-reports/)

> 2.7. Мы пишем код так, как будто у нас нет отдела QA
>
> Мы стараемся самостоятельно протестировать код. Если не знаем как его вызвать — узнаем. Если тестировать тяжело или долго — подробно в комментариях к задаче пишем инструкцию для QA. Это увеличит время “in development”, но время до production уменьшится: задача не зависнет в непонятных статусах.

Я пишу тесты сам и побуждаю писать тесты моих тиммейтов. Более того, я считаю, что юниттестов и интеграционных тестов много не бывает. Чтобы команда охотнее писала тесты, я предлагаю им забыть о том, что отдел тестирования отловит баги "если что". А если не отловит?

> 2.17. Читаемость важнее скорости и краткости
>
> Код гораздо чаще читают, чем пишут. Уделяем внимание форматированию, summary, описанию, README, понятным именам методов, переменных и классов.

Качество кода - вещь сложноизмеримая и сильно влияющая на скорость разработки и качество продукта. Компьютеры стали настолько мощные, что в приложениях, не предназначенных для высоких нагрузок (десятки или сотни тысяч запросов в секунду), нет необходимости заботиться о памяти и алгоритмической сложности так, как раньше. Лучше пусть будет код читаем и понятен любому, чем оптимальным и экономным по части памяти.

> 3.2. Если пишут по работе в выходные или после работы - можешь смело игнорировать
>
> Это нормально, если мы не отвечаем на сообщения в свои выходные дни или после окончания рабочего дня. Иногда автор даже и не ждет ответа, потому что письменное общение - это асинхронное общение. Мы имеем право ответить по таким вопросам в следующий рабочий день.

Моя личная боль. Я всегда нервничаю, когда получаю сообщения в рабочем мессенджере поздно вечером или выходной. Сразу представляю, что там продакшн упал и уже лихорадочно начинаю думать о том, какие личные дела можно отложить. А это, оказывается, коллега идеей поделился. Поэтому я не только настраиваю мьют уведомлений, но и рекомендую остальным это сделать и проговариваю вслух, что никто не обязан в выходные даже отвечать на такие сообщения.

> 3.3. Мы можем работать откуда угодно и когда угодно. Главное - результат
>
> У нас нет Core hours и мы можем работать вне офиса. Если в тикете есть дэдлайн, значит мы коммитаемся под него и делаем максимально возможное, чтобы выполнить к сроку задачу. Если мы не успеваем, то сообщаем команде об этом как можно раньше.

Формат работы уже сильно зависит от того, с чем связан проект, насколько команда разделена по часовым поясам и какой проектный менеджмент применяется. Сам я следил только за прогрессом по таскам, но я думаю, что декларирование Core Hours - когда в определенные часы по определенной таймзоне все участники команды должны быть на связи - это необходимость. Тогда каждый будет знать, что в эти часы ему ответят на сообщение, а в другие рассчитывать на это не придется. Главное - проговорить вслух и эту политику.

## Заключение

Если в вашей команде еще нет прописанных и опубликованных процессов, то скорее предлагайте тимлиду заняться этим. Если будет сопротивляться, то расскажите, что прописанным кодексом он сэкономит себе время на онбординг и ему не нужно будет отвечать на одни и те же вопросы забывчивым тиммейтам. Как в коде, так и в командной работе явное лучше неявного и очевидные вещи нужно проговаривать, чтобы у всех было одно видение.
