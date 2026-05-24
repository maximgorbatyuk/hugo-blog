---
layout: post
title: Андрей Карпатый о состоянии AI — пересказ интервью
category: different
tags: ["interview", "ai"]
date: "2026-05-24"
description: "Подробный пересказ интервью Андрея Карпатый (сооснователя OpenAI, бывшего руководителя Tesla Autopilot, автора термина «vibe coding») с ключевыми цитатами в его формулировках."
---

Это пересказ интервью Андрея Карпатого (Andrej Karpathy), опубликованного 29 апреля 2026 года на конференции AI Ascent 2026. Интервью в основном о том, что изменилось в агентской разработке за год с тех пор, как он придумал термин «vibe coding». Андрей объясняет, почему он никогда раньше не чувствовал себя настолько отстающим как программист, почему agentic engineering — более серьезная дисциплина, которая формируется поверх vibe coding, и почему о LLM стоит думать не как о "животных", которые могут обучаться в моменте, а как о призраках: неровных, статистических, вызываемых сущностях, которыми нужно управлять с новым уровнем вкуса и суждения. Он также говорит о Software 3.0, границах проверяемости и о том, почему можно отдать на аутсорсинг мышление, но не понимание.

Ссылка на оригинальное интервью: [Andrej Karpathy: From Vibe Coding to Agentic Engineering](https://youtu.be/96jN2OCOfLs?si=ULo7VgHMVVbD1DWh)

## Disclaimer

Статья подготовлена с помощью AI. Я разработал инструмент (набор скриптов с автоматизацией процесса), который берет на взод ссылку на подобное интервью и затем генерирует выжимку с ключевыми цитатами. Код доступен на GitHub - [maximgorbatyuk/video-tools](https://github.com/maximgorbatyuk/video-tools). Цель подобного инструмента - понять, стоит ли видео того, чтобы его посмотреть самому, я ни в коем случае не стремлюсь заменить просмотр оригинального материала быстрым прочтением пересказа. Перевод цитат на русский язык выполнен с помощью GPT5.5 с моей редакцией, и ниже каждой цитаты приведен оригинальный текст.

## Почему он чувствует себя «отстающим» программистом

Ощущение «отставания» появилось после настоящего переломного момента в декабре, когда AI-инструменты для программирования перешли для него важную границу. До этого модели выдавали фрагменты, которые нужно было редактировать. После этого они просто работали.

> «Я просто стал замечать, что с последними моделями фрагменты кода выходят приемлемыми. Я продолжал просить больше, и фрагменты продолжали на том же уровне. В какой-то момент я уже не мог вспомнить, когда в последний раз что-то исправлял. Я все больше и больше доверял системе, и так я начал vibe coding.»

> "I just started to notice that with the latest models, the chunks just came out fine. And then I kept asking for more and it just came out fine. And then I can't remember the last time I corrected it. And then I just trusted the system more and more and then I was vibe coding."

Он подчеркнул, что это не "едва заметное изменение": людям, которые пробовали AI-инструменты раньше в 2024 году и отбросили идею их применения, нужно посмотреть на них заново:

> «Многие в прошлом году воспринимали AI как ChatGPT или штуку с JSON, но вам действительно нужно было посмотреть еще раз, причем посмотреть по состоянию на декабрь (2025), потому что все фундаментально изменилось, особенно в построении взаимосвязанной работе агентов, которая действительно начала работать.»

> "A lot of people experienced AI last year as ChatGPT or JSON thing, but you really had to look again and you had to look as of December because things have changed fundamentally and especially on this like agentic coherent workflow that really started to actually work."

С тех пор, по его словам:

> «Папка с сайд-проектами забита множеством случайных вещей.»

> "side projects folder is extremely full with lots of random things."

## Software 3.0 как новая вычислительная парадигма

Он описал три эпохи: Software 1.0 = написание явного кода. Software 2.0 = обучение нейросетей (программирование через датасеты и архитектуры). Software 3.0 = промптинг LLM, где контекстное окно становится вашей управляющей поверхностью.

> «Software 3.0 в каком-то смысле означает, что ваше программирование теперь превращается в промптинг. А то, что находится в контекстном окне, — это ваш рычаг управления интерпретатором, то есть LLM, которая как бы интерпретирует ваш контекст и выполняет вычисления в цифровом информационном пространстве.»

> "Software 3.0 is kind of about your programming now turns to prompting. And what's in the context window is your lever over the interpreter, that is the LLM, that is kind of like interpreting your context and performing computation in the digital information space."

Он привел два примера, о которых постоянно думает. Первый: установщик OpenClaw — это просто блок текста, который вы вставляете своему агенту, а не shell-скрипт, который нужно скопировать и выполнить. Причина важна:

> «Теперь вы работаете в парадигме Software 3.0, где вам не нужно точно расписывать все отдельные детали этой настройки. У агента есть собственный разум, который следует инструкциям, смотрит на ваше окружение, ваш компьютер, и выполняет нужные действия, чтобы все заработало, а также отлаживает это все в цикле.»

> "You're working now in the software 3.0 paradigm where you don't have to precisely spell out all the individual details of that setup. The agent has its own intelligence that it packages up and then it kind of follows the instructions and it looks at your environment, your computer, and it kind of performs intelligent actions to make things work and it debugs things in the loop."

Второй, более радикальный пример: он сделал Menugen — приложение, которое берет фотографию ресторанного меню и генерирует изображения блюд. Потом он понял, что можно просто отдать фотографию Gemini с Nano Banana и получить отрендеренный результат напрямую, без приложения:

> «Весь мой Menugen лишний, он работает в старой парадигме — такого приложения не должно существовать. Парадигма Software 3.0 гораздо более сырая. Ваша нейросеть делает все больше и больше работы, а ваш промпт или контекст — это просто изображение, и выход — это изображение, и никакое приложение между ними не нужно.»

> "All of my Menugen is spurious, it's working in the old paradigm — that app shouldn't exist. The software 3.0 paradigm is a lot more raw. Your neural network is doing more and more of the work and your prompt or context is just the image and the output is an image and there's no need to have any of the app in between."

Его предупреждение: не стоит думать об LLM только как об ускорении того, что уже существует.

> «Сейчас это просто более обобщенная обработка информации, которую легче автоматизировать. Так что дело даже не только в коде… Появляются новые возможности для вещей, которые раньше были невозможны. И мне кажется, что это более интересно.»

> "This is more general information processing that is automatable now. So it's not just even about code… There's new opportunities of just things that couldn't be possible before. And I almost think that that's more exciting."

Его проект LLM-базы знаний — главный пример: превращение сырых документов в личную wiki — это то, что никакая традиционная программа не смогла бы сделать.

## Экстраполяция тренда

Он предложил гораздо более интересную долгосрочную картину. В 50-е и 60-е мы пошли по «калькуляторному» пути вычислений, но он считает, что путь нейросетей скоро начнет захватывать вышестоящий процесс:

> «Можно представить, что многое из этого перевернется. Нейросеть станет чем-то вроде хост-процесса. А CPU станет чем-то вроде сопроцессора.»

> "You could imagine that a lot of this will flip. The neural net becomes kind of like the host process. And the CPU has become kind of like the co-processor."

В таком будущем сырое видео или аудио попадает в модель, которая использует диффузию, чтобы отрендерить UI, настроенный под конкретный момент.

## Проверяемость и «неровный» интеллект

Он писал о том, почему LLM неравномерны: сильны в одних местах и безнадежны в других. Его теория в одну строку:

> «Традиционные компьютеры легко автоматизируют то, что вы можете описать в коде. А этот последний раунд LLM легко автоматизирует то, что вы можете проверить.»

> "Traditional computers can easily automate what you can specify in code. And kind of this latest round of LLMs can easily automate what you can verify."

Причина: передовые лаборатории обучают эти модели с помощью "подкрепляющего обучения" (reinforcement learning) в средах, где есть возможность проверяемости. Поэтому способности достигают пика в проверяемых доменах (математика, код, смежные области) и остаются грубыми во всем остальном. Его любимая иллюстрация «неровности»:

> «Я хочу поехать на автомойку, чтобы помыть машину, и она в 50 метрах от меня. Мне ехать или идти пешком? Современные модели сегодня скажут вам идти пешком, потому что это так близко. Как возможно, что Opus 4.7, который можно назвать произведением искусства, одновременно может отрефакторить кодовую базу на 100 000 строк или найти zero day уязвимости, но при этом говорит мне идти пешком до этой автомойки? Это безумие.»

> "I want to go to a car wash to wash my car and it's 50 meters away. Should I drive or should I walk? State-of-the-art models today will tell you to walk because it's so close. How is it possible that state of the art Opus 4.7 will simultaneously refactor a 100,000 line code base or find zero day vulnerabilities and yet tells me to walk to this car wash? This is insane."

Он хочет, чтобы люди усвоили два вывода. Первый: способности также зависят от того, что лаборатории решат скормить моделям. Он привел шахматы: там произошел огромный скачок от GPT-3.5 к GPT-4 не из-за чистого масштабирования, а потому что кто-то явно добавил шахматные данные в предобучение.

> «Мы немного зависим от того, что делают лаборатории, что именно они случайно добавят в смесь (данных). И вам нужно на самом деле исследовать эту штуку, которую они вам дают и у которой нет инструкции.»

> "We are slightly at the mercy of whatever the labs are doing, whatever they happen to put into the mix. And you have to actually explore this thing that they give you that has no manual."

Второй: ваше приложение находится в определенных «схемах», и вам нужно понять, в каких именно:

> «Если вы находитесь в схемах (circuits), которые были частью RL, вы на коне. А если вы работаете со схемой вне распределения данных, вам будет тяжело. И вам нужно понять, в каких схемах находится ваше приложение. А если вы не в нужных схемах, тогда вам действительно нужно смотреть в сторону тюнинга модели (fine tuning).»

> "If you're in the circuits that were part of the RL, you fly. And if you're in the circuits that are out of the data distribution, you're going to struggle. And you have to figure out which circuits you're in in your application. And if you're not in the circuits, then you have to really look at fine tuning."

## Совет фаундерам

Проверяемость — это не просто описание того, где LLM сегодня хороши. Это рычаг, которым фаундеры могут воспользоваться сами:

> «Если вы находитесь в проверяемой среде, где можете создать такие RL-среды или примеры, тогда это действительно дает вам возможность потенциально провести собственный тюнинг модели (fine tuning)… вы можете использовать свой любимый фреймворк, дернуть за "рычаг" и получить что-то, что на самом деле работает довольно хорошо.»

> "If you are in a verifiable setting where you could create these RL environments or examples, then that actually sets you up to potentially do your own fine tuning… you can use your favorite fine tuning framework and pull the lever and get something that actually works pretty well."

Он намекнул на конкретный недоиспользованный домен, но не назвал его со сцены. На более широкий вопрос о том, что «безопасно остается человеческим», его ответ был мрачным для тех, кто ставит на домены, защищенные от автоматизации:

> «В конечном счете почти все можно сделать проверяемым в той или иной степени. Даже для таких вещей, как письмо, можно представить совет LLM-судей и, вероятно, получить что-то разумное с помощью такого подхода. Все автоматизируемо.»

> "Ultimately almost everything can be made verifiable to some extent. Even for things like writing, you can imagine having a council of LLM judges and probably get something reasonable from this kind of an approach. Everything is automatable."

## Vibe coding против agentic engineering

В прошлом году он придумал термин «vibe coding», а теперь хочет четко отделить его от дисциплины, которая, по его мнению, появляется:

> «Vibe coding — это про повышение планки для всех в том, что они могут делать в программировании. Каждый может навайбкодить что угодно. Это потрясающе. Но агентская разработка — это про сохранение планки качества, которая раньше существовала в профессиональных приложениях. Вам нельзя игнорировать уязвимости просто потому что вы навайбкодили это. Вы по-прежнему отвечаете за свое приложение, как и раньше.»

> "Vibe coding is about raising the floor for everyone in terms of what they can do in software. Everyone can vibe code anything. That's amazing. But agentic engineering is about preserving the quality bar of what existed before in professional software. You're not allowed to introduce vulnerabilities due to vibe coding. You are still responsible for your software just as before."

И он считает, что потолок продуктивности здесь огромен:

> «Раньше люди говорили о 10х engineer. 10х — это не про скорость, которую вы получаете. Люди, которые очень хорошо работают нейросетями получают гораздо больше, чем 10х.»

> "People used to talk about the 10X engineer previously. 10X is not the speed up you gain. People who are very good at this peak a lot more than 10X from my perspective right now."

Его раздражает то, как компании нанимают:

> «Большинство людей все еще не перестроили свой процесс найма под контекст агентской разработки. Если вы даете задачи для решения (во время интервью), это все еще старая парадигма. Найм должен выглядеть так: дайте мне действительно большой проект и посмотрите, как человек реализует что-то в этом большом проекте.»

> "Most people are still not refactored their hiring process for agentic engineer capability. If you're giving out puzzles to solve, this is still the old paradigm. Hiring has to look like: give me a really big project and see someone implement in that big project."

Его конкретный пример: построить безопасный клон Twitter для агентов, задеплоить его, а затем запустить злоумышленных агентов (adversarial agents) (например, «10 codex 5.4 X high»), чтобы попытаться его сломать.

## Что все еще остается за людьми

Его честная реакция на чтение кода, написанного агентами:

> «Когда ты на самом деле смотришь на код, у меня иногда почти случается сердечный приступ, потому что это не всегда суперпотрясающий код. Он очень раздутый, в нем много copy paste, есть неловкие абстракции, которые хрупкие, и он работает, но он просто очень отвратительный.»

> "When you actually look at the code, sometimes I get a little bit of a heart attack because it's not like super amazing code necessarily all the time. It's very bloaty and there's a lot of copy paste and there's awkward abstractions that are brittle and it works, but it's just really gross."

Так что вкус, дизайн и написание спецификаций пока остаются человеческой работой. Он привел отличный конкретный failure mode из Menugen: его агент сопоставлял пользователей между Stripe и Google по *email address*, а не по стабильному user ID. Email может измениться; это привело бы к "беззвучной" потере денег людей.

> «Вы отвечаете за идею, разработку, дизайн и за то, чтобы все имело смысл и чтобы вы просили нужные вещи… вы делаете проектирование и наработки, а инженеры [имеются в виду агенты] заполняют пробелы.»

> "You're in charge of the taste, the engineering, the design, and that it makes sense and that you're asking for the right things… you're doing some of the design and development and the engineers [meaning the agents] are doing the fill in the blanks."

А вот вещи, о которых он больше не заботится:

> «Я уже забыл все эти нюансы: keep dims или keep dim, dim это или axis, reshape, permute или transpose. Я больше не держу это в голове, потому что в этом уже нет необходимости. Это как раз та мелочь, которую можно поручить стажёру — у него отличная память.»

> "I already forgot about the keep dims versus keep dim or whether it's dim or axis or reshape or permute or transpose. I don't remember this stuff anymore because you don't have to. This is the kind of details that are handled by the intern because they have very good recall."

Когда его спросили, будет ли вкусовщина со временем иметь меньшее значение, он ответил честно: вероятно, все улучшится, когда лаборатории добавят эстетические награды в RL:

> «Но лаборатории пока почти этого не сделали.»

> "but the labs haven't done it yet almost."

## Призраки, а не животные

Философская рамка, которую он обдумывает: эти штуки были сформированы не эволюцией, любопытством или развлечением. Их нельзя сравнить с животными.

> «Эти штуки не являются животным интеллектом. Если вы кричите на агентов, они не станут работать лучше или хуже — это никак не влияет. Все это просто что-то вроде статистических симуляционных схем, где суть — это предобученная модель (pre-training). То есть статистика. А потом сверху прикручивается RL.»

> "These things are not animal intelligences. If you yell at them, they're not going to work better or worse — it doesn't have any impact. It's all just kind of like these statistical simulation circuits where the substrate is pre-training. So like statistics. And then there's RL bolting on top."

Он признает, что это отчасти «философствование», но эта рамка формирует подход к работе с ними: подозрительность и постепенное исследование вместо антропоморфизации.

## Agent-native инфраструктура

То, что его особенно раздражает: документация все еще пишется для людей.

> «Каждый раз, когда мне говорят “перейди по этому URL”, я такой: ох. Почему люди все еще говорят мне, что делать? Я не хочу ничего делать. Что именно я должен скопировать и вставить своему агенту?»

> "Every time I'm told 'go to this URL,' it's just like, oh. Why are people still telling me what to do? I don't want to do anything. What is the thing I should copy-paste to my agent?"

Его тест на то, становится ли инфраструктура agent-native: когда он писал Menugen, писать код было легко, а деплоить его на Vercel и связывать сервисы было болью.

> «Я бы хотел иметь возможность дать LLM промпт: построить Menugen, и чтобы мне не пришлось ничего трогать, а оно было задеплоено в интернете тем же образом. Думаю, это был бы хороший своего рода тест того, становится ли значительная часть нашей инфраструктуры все более agent native.»

> "I would hope that I could give a prompt to an LLM, build Menugen, and that I didn't have to touch anything and it's deployed in that same way on the internet. I think that would be a good kind of a test for whether or not a lot of our infrastructure is becoming more and more agent native."

Долгосрочная картина:

> «Мы идем к миру, где у людей и организаций есть агентное представительство. Мой агент будет говорить с вашим агентом, чтобы выяснить часть деталей наших встреч.»

> "We're going towards a world where there's agent representation for people and for organizations. I'll have my agent talk to your agent to figure out some of the details of our meetings."

## Финал — об образовании и понимании

Финал был самой сильной мыслью интервью. Твит, который ему запомнился:

> «Вы можете отдать свое мышление на аутсорсинг, но вы не можете отдать на аутсорсинг свое понимание.»

> "You can outsource your thinking, but you can't outsource your understanding."

Его объяснение, почему это все еще важно:

> «Я становлюсь бутылочным горлышком даже в том, чтобы понимать, что мы пытаемся построить. Почему это стоит делать? Как мне направлять моих агентов? Что-то должно направлять мышление и обработку, и это все еще как бы фундаментально ограничено пониманием.»

> "I'm becoming a bottleneck of just even knowing what are we trying to build. Why is it worth doing? How do I direct my agents? Something has to direct the thinking and the processing, and that's still kind of fundamentally constrained by understanding."

Именно поэтому его так радуют личные базы знаний, построенные LLM: это инструменты для *обработки* информации, а не замена ей.

> «Каждый раз, когда я вижу другую проекцию на информацию, я всегда чувствую, что получаю инсайт. Это инструменты, которые определенным образом усиливают понимание. И это все еще своего рода бутылочное горлышко, потому что вы не можете быть хорошим директором, если [вы не понимаете], а LLM, конечно, не особенно хороши в понимании. Вы по-прежнему уникально отвечаете за это.»

> "Anytime I see a different projection onto information, I always feel like I gain insight. These are tools to enhance understanding in a certain way. And this is still kind of a bit of a bottleneck because you can't be a good director if [you don't understand] — the LLMs certainly don't excel at understanding. You still are uniquely in charge of that."

Он закончил тем, что ему было бы интересно вернуться через пару лет и посмотреть, закрыли ли агенты даже этот разрыв.