---
layout: post
title: "Вредные советы по тестированию программ"
category: development
tags: [.net, unit_testing, testing, anti_patterns]
date: "2023-03-04"
description: "Тестировать программы, которые пишем - долго, нудно и муторно. Поди разбери, что хочет тимлид. Хорошо, что люди придумали много антипаттернов, как можно и юнит-тесты написать, и команду свою запутать. Вроде тесты написаны, а что проверяют и как - не поймешь и не разберешь. В статье ниже я дам вам несколько вредных советов, как можно пошутить над тиммейтами с помощью юнит-тестов."
draft: false
---

> Disclaimer:
>
> 1. Советы ниже - __вредные__, доверять им не стоит.
>
> 2. Антипаттерны придуманы не мной. В статье я стараюсь дать примеры, на которых будет видно, почему такие подходы и назвали антипаттернами
>
> 3. Код юнит-тестов - это не second-class код, его тоже нужно писать поддерживаемым, читаемым и понятным для остальных ребят в команде.
>
> 4. Курсивом - вредный совет, обычным шрифтов - пояснение.

## 0. Кукушка (Cuckoo, aka Stranger)

_Твой класс делает запросы к внешним системам? Отлично, проверь и вызовы тоже вместе с выходными данными. Кому хуже будет?_

![Cuckoo!](/images/blog/development/2023-03-04/cuckoo-jack-black.gif)

Представим, что у нас есть сервис пользователей, который делает запросы на внешний API:

```typescript
// src/user-service.ts
import axios from 'axios';

export interface User {
    id: string;
    name: string;
}

export class UserService {
    async get(userId: string): Promise<User> {
        const response = await axios.get(`https://api.example.com/users/${userId}`);
        return response.data;
    }
}

```

```typescript
// tests/user-service.test.ts
import chai from 'chai';
import { expect } from 'chai';
import * as sinon from 'sinon';
import sinonChai from 'sinon-chai';
import axios from 'axios';
import { UserService } from '../src/user-service';

describe('UserService', () => {

    chai.use(sinonChai);
    let axiosGetStub: sinon.SinonStub;

    beforeEach(() => {
        axiosGetStub = sinon.stub(axios, 'get');
    });

    afterEach(() => {
        axiosGetStub.restore();
    });

    it('should fetch user data using axios', async () => {

        const userId = "random-uuid";
        const fakeData = { id: userId, name: 'John Doe' };
        axiosGetStub.resolves({ data: fakeData });

        const result = await new UserService().get(userId);

        // Assertion
        expect(axiosGetStub).to.have.been.calledOnceWith(`https://api.example.com/users/${userId}`);
        expect(result).to.equal(fakeData);
    });
})
```

В примере мы проверяем не только выходные данные на соответствие ожидаемому результату, но и факт, что был вызван `axios.get` с правильными параметрами и урлом. Кажется, что все корректно, однако внутренняя логика работы метода и вызовы внешних сервисов должна быть ответственностью самого класса, а мы лишь проверяем аутпут метода. Класс может поменять вызовы, урлы, параметры, но все так же будет отдавать инстанс юзера. В этом случае тест станет красным, а разработчик не сразу поймет, что нужно делать: логику чинить или тест исправлять.

Уберите проверку axios, и тогда тест станет более надежным:

```typescript
// tests/user-service.test.ts
it('should fetch user data using axios', async () => {
    // Arrange
    // ...

    // Act
    // ...

    // Assertion
    expect(result).to.equal(fakeData);
});
```

Есть еще одна вариация этого антипаттерна - ["Forty-Foot Pole"](https://stackoverflow.com/a/339247). Так называют тесты, проверяющие функционал тех классов, которые лежат не за одним слоем абстракции глубоко внутри.

## 1. Один тест на каждый метод (test-per-method)

_Тимлид просит тебя покрыть тестами новую фичу? Начни с тестов на каждый метод новых классов. А как начал, так и закончи. Тесты есть? Есть! Хватит их писать, достаточно, там еще задачи на доске есть._

![Stop](/images/blog/development/2023-03-04/stop_sign.jpeg)

Иметь по одному тесту на каждый метод рабочего кода - это хорошее начало, но так функционал системы и пограничные кейсы не проверишь.

```typescript
// src/testable-class.ts
export class TestableClass {
    makeSomeLogic1(): number {
        // some logic goes here
        return 42;
    }

    makeSomeLogic2(): number {
        // some logic goes here
        return 43;
    }

    makeSomeLogic3(): number {
        // some logic goes here
        return 44;
    }
}
```

```typescript
// tests/testable-class.test.ts
import { TestableClass } from './anal-probe-example';

describe('TestableClass', () => {
    it('makeSomeLogic1 returns 42', () => {
        const target = new TestableClass();
        const result = target.makeSomeLogic1();
        expect(result).toBe(42);
    });

    it('makeSomeLogic2 returns 43', () => {
        const target = new TestableClass();
        const result = target.makeSomeLogic2();
        expect(result).toBe(43);
    });

    it('makeSomeLogic3 returns 44', () => {
        const target = new TestableClass();
        const result = target.makeSomeLogic3();
        expect(result).toBe(44);
    });
});
```

Не останавливайся на первом шагу. Нужно проверять дальше пограничные кейсы и ошибочные сценарии, и тогда код будет более надежным, а ошибки будут найдены гораздо раньше, чем это сделают конечные пользователи системы.

## 2. Анальная пробка (Anal probe)

_Архитектура классов неудобная, куча полей закрыто, но только по ним можно понять результат работы логики? Не беда, рефлексия или хаки помогут тебе проверить то, что скрыто внутри! Погугли, как можно проверять закрытые свойства класса в твоем ЯП и скорее добавь в тест ассерты._

![Censored](/images/blog/development/2023-03-04/censored.png)

Тестирование приватных полей, методов. Тест знает слишком много о тестируемом классе и лезет не в свое дело. Покажу на примере. Тут у нас тестируемый класс, который изменяет приватное поле:

```typescript
// src/testable-class.ts
export class TestableClass {
    constructor(private value: string) {}

    public makeSomeLogic(): void {
        this.value += ', 1';
    }
}
```

```typescript
// tests/testable-class.test.ts
import { TestableClass } from './anal-probe-example';

describe('TestableClass', () => {
    it('value should be changed', () => {
        const target = new TestableClass('example');

        target.makeSomeLogic();
        expect((target as any).value).toBe('example, 1');
    });
});
```

Анальной пробкой называют тесты, которые слишком много знают о классе, который тестируют, и лезут не в свое дело. В данном случае тест проверяет значение приватного поля. А что делать, если приватное поле уберут, а логика класса в целом не поменялась? Тест будет красным, придется переписывать.

У классов и интерфейсов системы не зря есть публичные свойства и методы - они и предназначены для того, чтобы с их помощью вызывать нужный функционал. Обычно библиотеки разрабатывают так, чтобы внешне система не меняла свой интерфейс, но внутренне она становилась только лучше. Такой подход стоит применить и в коде своего проекта тоже: выставляйте наружу публичный относительно статичный интерфейс, а в тестах проверяйте результаты выходных данных из этого интерфейса.

## 3. Happy path

_Написал фичу, и тимлид требует доказать ее работоспособность? Отлично, сейчас он получит свое! Напиши один тест, проверяющий только один рабочий сценарий использования, и отправляй смело на ревью. Просил доказать, ты доказал. Что не так?_

![Happy pepe](/images/blog/development/2023-03-04/pepe-clap.gif)

Проблема таких тестов в том, что они проверяют только прохождение "счастливого пути", не затрагивая пограничные кейсы и ошибочные сценарии. Например, у нас есть класс проверки возраста и его тест-класс:

```typescript
// src/age-detector.ts
export class AgeDetector {
    constructor(private readonly yearOfBirth: number) {}

    howOldAmI(): number {
        const currentYear = new Date().getFullYear();
        return currentYear - this.yearOfBirth;
    }
}
```

```typescript
// tests/age-detector.test.ts
import { AgeDetector } from './business-service';

describe('AgeDetector', () => {
    it('howOldAmI returns 30', () => {
        const target = new AgeDetector(1993);

        const result = target.howOldAmI();
        expect(result).toBe(30);
    });
});
```

В тесте мы проверили, что класс возвращает разницу. Но что насчет кейсов, когда дата рождения в этом году еще не наступила? А если передали дату в будущем в конструктор класса? А если передали отрицательное число?

```typescript
// tests/age-detector.test.ts
describe('AgeDetector', () => {
    it('howOldAmI returns 30', () => {
        const target = new AgeDetector(1993);

        const result = target.howOldAmI();
        expect(result).toBe(30);
    });

    it('ctor throws Error if date is in future', () => {
        expect(new AgeDetector(2050)).toThrowError();
    });

    it('ctor throws Error if date is in the past far from now', () => {
        expect(new AgeDetector(1850)).toThrowError();
    });
});
```

Таких пограничных кейсов может быть тысячи, поэтому не останавливайся в написании тестов на только лишь одном успешном сценарии.

> Чем больше ошибок отловишь ты и закрепишь их в тестах, тем надежнее будет твой код.

## 4. Слоупок (Slowpoke)

_В проекте уйма тестов, а темп работы слишком быстрый для тебя? Не беда, есть способ добавить себе пару свободных часов в неделю. Напиши в тестах вызовы внешних сервисов, добавь ожидания ответов, записи файлов. Так тесты будут выполняться дольше, а у тебя появятся свободные полчаса каждый раз, когда отправляешь свой код на тестовый стенд для прогона пайплайна._

![Slowpoke](/images/blog/development/2023-03-04/slowpoke.jpg)

Бывают тесты, которые воспроизводятся очень долго. Часто тесты медленные, если они работают с внешними системами, делают запросы в другие сервисы или взаимодействуют с файловой системой. Такие тесты ненадежны и по другим причинам, но в данном кейсе они еще и увеличивают общее время выполнения пайплайна.

```typescript
// src/weather-service.ts
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable, map, catchError } from 'rxjs';

export interface WeatherData {
    main: {
      temp: number;
    };
    weather: {
      description: string;
    }[];
  }

@Injectable({
    providedIn: 'root'
})
export class WeatherService {

    private readonly apiKey: string;
    private readonly baseUrl: string;

    constructor(
        private readonly http: HttpClient) {
        this.apiKey = '1234567890';
        this.baseUrl = 'https://api.openweathermap.org/data/2.5/weather';
    }

  getWeather(city: string): Observable<string> {
    const url = `${this.baseUrl}?q=${city}&appid=${this.apiKey}&units=metric`;

    return this.http.get<WeatherData>(url).pipe(
      map(response => {
        const temperature = response.main.temp;
        const description = response.weather[0].description;
        return `Temperature in ${city}: ${temperature}°C, ${description}`;
      }),
      catchError((error: HttpErrorResponse) => {
        return `Error getting weather for ${city}. Status: ${error.status}`;
      })
    );
  }
}
```

И тест для этого сервиса:

```typescript
// tests/weather-service.test.ts
import { TestBed } from '@angular/core/testing';
import { WeatherService } from './business-service';
import { HttpClientModule } from '@angular/common/http';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('WeatherService', () => {

    let httpMock: HttpTestingController;

    beforeEach(() => {
        TestBed.configureTestingModule({
            imports: [HttpClientModule, HttpClientTestingModule],
            providers: [WeatherService]
        });

        httpMock = TestBed.inject(HttpTestingController);
    });

    afterEach(() => {
        httpMock.verify();
    });

    it('should return weather information for a city', () => {
        const city = 'London';
        const target = TestBed.inject(WeatherService);

        target.getWeather(city).subscribe(response => {
            expect(response).toContain(`Temperature in ${city}:`);
        });
    });
});
```

Если тестов, которые зависят от внешних факторов, много, то и время исполнения будет рандомным и зависеть от состояния нагруженности сети, зависеть от погоды, etc. Тесты пишем так, чтобы они проверяли логику работы на основе заранее определенного ответа внешнего сервиса. Для этого нужно мокать, подменять сервисы заглушками и рефакторить. Пример с моком внешнего сервиса ниже:

```typescript
// tests/seather-service.test.ts
import { TestBed } from '@angular/core/testing';
import { WeatherData, WeatherService } from './business-service';
import { HttpClientModule } from '@angular/common/http';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('WeatherService', () => {

    let httpMock: HttpTestingController;

    beforeEach(() => {
        TestBed.configureTestingModule({
            imports: [HttpClientModule, HttpClientTestingModule],
            providers: [WeatherService]
        });

        httpMock = TestBed.inject(HttpTestingController);
    });

    afterEach(() => {
        httpMock.verify();
    });

    it('should return weather information for a city', () => {
        const city = 'London';
        const apiKey = '1234567890';
        const target = TestBed.inject(WeatherService);

        target.getWeather(city).subscribe(response => {
            expect(response).toEqual(`Temperature in ${city}: 15°C, cloudy`);
        });

        const request = httpMock.expectOne(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`);
        expect(request.request.method).toBe('GET');
        request.flush({
            main: { temp: 15 },
            weather: [{ description: 'cloudy' }]
        } as WeatherData);
    });

    it('should handle errors', () => {
        const city = 'InvalidCity';
        const apiKey = '1234567890';

        const target = TestBed.inject(WeatherService);

        target.getWeather(city).subscribe(response => {
            expect(response).toEqual(`Error getting weather for InvalidCity. Status: 404`);
        });

        const request = httpMock.expectOne(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`);
        expect(request.request.method).toBe('GET');
        request.flush('', { status: 404, statusText: 'Not Found' });
    });
});
```

## 5. Гигант (Giant)

_Зачем тебе нужно добавлять новые тестфайлы, писать новый код, который подготавливает условия для выполнения тестов, что-то самому сочинять? Есть же вон тестфайл готовый, добавь в него еще пару тестов!_

![Giant unittest](/images/blog/development/2023-03-04/long_unit-test.png)

Если тест-файл содержит в себе тысячи строк кода, то причиной может быть ситуация, когда тестируемый класс содержит очень много логики и ответственности. Тут и одной тысячей строк кода не обойтись. Что делать в этом случае? Только дробить и класс, и файл тестов соответственно.

![1k lines of test](/images/blog/development/2023-03-04/1_k_line_of_tests.png)

![9k lines of logic](/images/blog/development/2023-03-04/over9000.png)

Дробите большие классы, устраняйте god objects из своего проекта. В куче строк кода легко потеряться и допустить ошибки.

## 6. Mockery

_Моки - это круто! Как хочешь, так и подменяй логику. Применяй моки везде и всюду, ведь это единственно верный подход. В классе единственная зависимость и нет бизнес-логики? Все равно! Применяй моки!_

![Mockery](/images/blog/development/2023-03-04/mockery.jpeg)

Моки - это отличный способ подменить функциональность. Так можно протестировать свою бизнес-логику, которая зависит на ответе стороннего сервиса, и при эётом не вызывать этот сторонний сервис. Но и перебарщивать с этим не стоит, иначе можно столкнуться с ситуацией, когда вы мокаете сервисы, а потом проверяете работу этих же моков. Нужно соблюдать баланс.

Например, на бэкенде популярен паттерн "репозиторий" для создания прослойки между бизнес-логикой и хранилищем данных. Ниже можно увидеть, как мы настраиваем с помощью моков класс, но внутри класса логики-то толком и нет

![Setup mocks for testclass](/images/blog/development/2023-03-04/mock_dotnet.png)

А ниже - сам класс. Логики внутри почти нет, в итоге тестировать и нечего особо.

![Setup mocks for testclass](/images/blog/development/2023-03-04/class_under_tests_dotnet.png)

Даже такие классы нужно тестировать, но в данном случае лучше подойдет мок всей БД, куда сначала мы кладем искомый объект, а потом уже проверяем, что он будет возвращен бизнес-логикой.

## 7. Инспектор (Inspector)

_Ты - тимлид и хочешь помериться долей покрытия тестами с другими тимлидами в чате? тлично, так и нужно делать! Поставь задачу своим ребятам, чтоб обеспечили только 100% code-coverage. А какой ценой? Не имеет значения!_

![What did it cost?](/images/blog/development/2023-03-04/what_did_it_cost.png)

В погоне за метрикой покрытия тестами можно так увлечься, что смысл этой метрики будет потерян. Вследствие этого появляются тесты, которые проверяют настолько глубокий функционал внутри класса, что при любом рефакторинге нужно переписывать и тест, даже если логика не поменялась. Это плохо, потому что когда разработчики видят, что тест упал и при это внешний интерфейс класса не был изменен, они пытаются сразу понять, что именно в бизнес-логике они сломали. И как же обидно понять, что тест сломался не потому, что ты что-то сломал, а потому, что ты просто переименовал переменную внутри класса.

Этот антипаттерн схож с "анальной пробкой", но у него иная цель: если "анальная пробка" преследует цель проверить результат работы логики по скрытым полям класса, то тут целью ставят полное покрытие кода тестами, а качество тестов значения не имеет.

Есть открытый [вопрос на SO](https://stackoverflow.com/q/4520216), заданный Егором Бугаенко, о том, как написать тест для приватного конструктора класса в Java. Вопрос был задан скорее для того, чтобы собрать мнение сообщества по вопросу, и [один из лучших ответов](https://stackoverflow.com/a/4520241) я приведу тут:

> Remember that coverage is meant to be something which is useful to you - you should be in charge of the tool, not the other way round.

## 8. Generous Leftovers (aka Chain Gang, Wet Floor)

_В проекте нужно написать тесты на генерацию CSV и парсинг этого CSV? Отлично, можно же убить двух зайцев! Напиши один тест на генерацию файла, а второй - на его чтение и парсинг. Тимлид точно оценит твою идею._

![Brugge](/images/blog/development/2023-03-04/brugge.jpeg)

Представим пример двух тестов, один из которых проверяет создание CSV файла, а второй - читает его.

```typescript
// test/createFile.test.ts
import { expect } from 'chai';
import { promises as fs } from 'fs';
import path from 'path';

describe('File creation', () => {
    const filePath = path.join(__dirname, 'testFile.txt');
    const fileContent = 'Hello, world!';

    it('should create a new file', async () => {
        await fs.writeFile(filePath, fileContent);
        const fileExists = await fs.stat(filePath).then(() => true).catch(() => false);
        expect(fileExists).to.be.true;
    });
});
```

```typescript
// tests/readFile.test.ts
import { expect } from 'chai';
import { promises as fs } from 'fs';
import path from 'path';

describe('Read and process file', () => {
    const filePath = path.join(__dirname, 'testFile.txt');
    const fileContent = 'Hello, world!';

    it('should read the file and capitalize the content', async () => {
        const content = await fs.readFile(filePath, 'utf-8');
        const capitalizedContent = content.toUpperCase();
        expect(capitalizedContent).to.equal('HELLO, WORLD!');
    });
});
```

Вывод в консоль:

```bash
File creation
    ✔ should create a new file

Read and process file
    ✔ should read the file and capitalize the content

2 passing (4ms)
```

Тимлид не оценит такие тесты. Этот подход плохой, потому что:

- возникает необходимость запускать некоторые тесты в строгом порядке,
- возникает зависимость тестов друг от друга, они не атомарны.

Лучше создавать файл перед тестом и удалять его после. Так тест будет атомарным и независимым, хоть выполнение будет занимать больше времени

```typescript
// tests/readFile.test.ts
import { expect } from 'chai';
import { promises as fs } from 'fs';
import path from 'path';

describe('Read and process file', () => {
    const filePath = path.join(__dirname, 'testFile.txt');
    const fileContent = 'Hello, world!';

    // Creating file before the test
    before(async () => {
        await fs.writeFile(filePath, fileContent);
    });

    it('should read the file and capitalize the content', async () => {
        const content = await fs.readFile(filePath, 'utf-8');
        const capitalizedContent = content.toUpperCase();
        expect(capitalizedContent).to.equal('HELLO, WORLD!');
    });

    // Removing file after the test
    after(async () => {
        await fs.unlink(filePath);
    });
});
```

## 9. Local Hero (aka "Скрытая зависимость", "Евангелист ОС", "Хулиган окружения")

_Тимлид заставляет написать тест на класс, в котором используются переменные окружения? Не беда! Пиши смело так, чтобы проходили на сервере CI/CD. НА локальном окружении фейлятся? Да какая разница, гдавное - МР пройдет!_

![Captain America](/images/blog/development/2023-03-04/captain_america.png)

```typescript
// src/app-storage.ts
interface User {
    id: number;
    name: string;
}

export class AppStorage {
    private readonly _storage: Array<User> = [];

    seed(): void {
        const database = [];
        if (process.env.ENVIRONMENT === 'Development') {
            this._storage.push({
                id: 1,
                name: 'John Doe',
            });
        }
    }

    count(): number {
        return this._storage.length;
    }
}
```

```typescript
// tests/app-storage.test.ts
import { expect } from 'chai';
import { AppStorage } from '../src/app-storage';

describe('AppStorage', () => {

    let target: AppStorage;

    before(async () => {
        target = new AppStorage();
    });

    it('should seed data in development', async () => {

        // The test will be green on CI/CD server
        // because it has process.env.ENVIRONMENT = 'Development';
        target.seed();
        expect(target.count()).to.equal(1);
    });
});
```

Делайте свои тесты независимыми от окружения и внешних обстоятельств максимально. Если логика зависит от переменных окружения, то напишите класс-провайдер этих переменных, и тогда вы сможете его замокать в тестах:

```typescript
// src/app-storage.ts
export interface IEnvironment {
    ENVIRONMENT: string;
}

interface User {
    id: number;
    name: string;
}

export class AppStorage {
    private readonly _storage: Array<User> = [];
    constructor(private readonly env: IEnvironment){}

    seed(): void {
        if (this.env.ENVIRONMENT === 'Development') {
            this._storage.push({
                id: 1,
                name: 'John Doe',
            });
        }
    }

    count(): number {
        return this._storage.length;
    }
}
```

```typescript
import { expect } from 'chai';
import { AppStorage, IEnvironment } from '../src/app-storage';

describe('AppStorage', () => {

    it('should seed data in development', async () => {

        const env = {
            ENVIRONMENT: 'Development',
        } as IEnvironment;

        const target = new AppStorage(env);
        target.seed();
        expect(target.count()).to.equal(1);
    });
});
```

Автору встречались тесты, котоые фейлились в зависимости от "культуры", установленной на компьютере: на ноуте автора с английской культурой тесты были зелеными, но у коллег с немецкой они фейлились. Причиной была разница в способе форматирования чисел с дробью: в английской культуре применялась точка, а в немецкой - запятая. Беда была еще в том, что билд-сервер тоже был с английской культурой и не показал, что тесты фейлятся. Решили проблему легко - явно форматировали числа с дробью в тестах с использованием точки.

## 10. Придира (Nitpicker)

_Заставляют писать тесты даже на выгрузку CSV? Ну что ж, отомсти им, сделай ассерт на весь контент! Пусть в случае ошибки помучаются искать, что именно не соответствует ожиданию._

"Придирой" называют тесты, которые проверяю лишь незначительную часть выходных данных тестируемого метода. Например, клас формирует репорт большого размера, и мы проверяем весь выходной текст на соответствие ожидаемому результату. Казалось бы, такое поведение корректно, однако в случае ошибки в одной из колонок мы получим сложночитаемую ошибку в логах выполнения тестов.

```typescript
// src/csv-exporter.ts
export interface Person {
    id: number;
    name: string;
    email: string;
    dateOfBirth: string;
}

export class CSVExporter {
    constructor(private readonly data: Person[]) {}

    private escapeCSVField(field: string): string {
        return `"${field.replace(/"/g, '""')}"`;
    }

    csv(): string {
        const header = 'id,name,email,dateOfBirth';
        const rows = this.data.map(person => {
            const fields = [
            person.id,
            this.escapeCSVField(person.name),
            this.escapeCSVField(person.email),
            person.dateOfBirth,
            ];
            return fields.join(',');
        });

        return [header, ...rows].join('\n');
    }
}
```

```typescript
import { expect } from 'chai';
import { CSVExporter } from '../src/csv-exporter';

describe('File creation', () => {

    it('should create a new file', async () => {

        const people = [
        {
            id: 1,
            name: 'John Doe',
            email: 'john.doe@example.com',
            dateOfBirth: '1990-01-01',
        },
        {
            id: 2,
            name: 'Jane Doe',
            email: 'jane.doe@example.com',
            dateOfBirth: '1992-02-02',
        },
        {
            id: 3,
            name: 'Jim Doe',
            email: 'jim.doe@example.com',
            dateOfBirth: '1993-02-02',
        },
        ];

        const target = new CSVExporter(people);
        const output = target.csv();

        const expected = `id,name,email,dateOfBirth
        1,"John Doe","john.doe@example.com",1990-01-01
        2,"Jane Doe","jane.doe@example,com",1992-02-02
        3,"Jim Doe","jim.doe@example.com",1993-02-02`;

        expect(output).to.equal(expected);
    });
});
```

```bash
1) CSVExporter
should return proper csv:

    AssertionError: expected 'id,name,email,dateOfBirth\n1,"John Do…' to equal 'id,name,email,dateOfBirth\n1,"John Do…'
    + expected - actual

    id,name,email,dateOfBirth
    1,"John Doe","john.doe@example.com",1990-01-01
    -2,"Jane Doe","jane.doe@example.com",1992-02-02
    +2,"Jane Doe","jane.doe@example,com",1992-02-02
    3,"Jim Doe","jim.doe@example.com",1993-02-02
```

Ошибка во второй строке полученного CSV - в емейле вместо точки поставлена запятая. Тест показал, что ожидаемый результат не совпадает, но вот не сразу получается разглядет причину ошибки. Старайтесь валидировать частями или разбивать на несколько тесткейсов подобные случаи, чтобы в случае ошибок сразу видеть их причину.

## 11. Шкатулка с секретом (Secret Catcher)

_Нужно написать тест на выбрасывание ошибки? Отлично, тесты ведь  нужны для этого! Пусть один из тестов будет красным, ведь мы ждем ошибку там. Главное - не забудь комментарий написать, что фйл теста - это ожидаемо._

![The secret box](/images/blog/development/2023-03-04/secret_box_sponge.png)

На первый взгляд, такой тест не делает ничего, ведь в нем нет ассертов. Однако дьявол кроется в деталях. На самом деле, тест надеется, что внутри произойдет эксепшн, и консоль покажет "ожидаемый" текст ошибки в логах.

```typescript
// src/secret-catcher-service.ts
export class SecretCatcherService {
    constructor(
    private someCondition = false) {}

    doSomeLogic(): void {
        // do some logic
        // ...
        // ...
        // probably, we change someCondition to true

        if (this.someCondition) {
            throw Error('Logic error!');
        }
    }
}
```

```typescript
// tests/secret-catcher-service.test.ts
import { expect } from 'chai';
import { SecretCatcherService } from '../src/secret-catcher-service';

describe('SecretCatcherService', () => {
    it('should not throw an error if we pass false', () => {
        const target = new SecretCatcherService(false);

        // red test is ok!
        target.doSomeLogic();
    });
});
```

Лучшим выходом будет отлавливать явно эксепшн и проверять его поля на ожидаемый текст ошибки, например, или иные ее поля. Современные тест-фреймворки могут отлавливать ошибки, поэтому используйте их возможности.

```typescript
// tests/secret-catcher-service.test.ts
import { expect } from 'chai';
import { SecretCatcherService } from '../src/secret-catcher-service';

describe('SecretCatcherService', () => {
    it('should throw an error if we pass true', () => {
        const target = new SecretCatcherService(true);

        expect(() => target.doSomeLogic()).to.throw('Logic error!');
    });
});
```

## 12. Уклонист (Dodger)

_Пишешь тесты, но внезапно оказалось, что часть логики зависит от внешних сервисов. И никак не замокать. Есть выход! Добавь побольше побочных тестов, особенно если они проверяют маловероятные случаи. Тимлид будет доволен и аппрувнет МР._

"Уклонистом" можно назвать тест, который проверяет маловероятные кейсы, не оказывающие значительное влияние на поведение класса, но при этом как будто бы специально пропускает важные тесткейсы. Пример привести сложно, ведь тут все сильно зависит от бизнеса, для которого работает приложение - бизнес определяет, какие кейсы важные и какие - маловероятные.

```typescript
// src/division.ts
export class Division {
    constructor(
    private readonly a: number,
    private readonly b: number) {}

    result(): number {
        if (this.b === 0) {
            throw new Error('Division by zero!');
        }

        return this.a / this.b;
    }
}
```

```typescript
// tests/division.test.ts
import { expect } from 'chai';
import { Division } from '../src/division';

describe('Division', () => {
    it('should divide positive numbers. Returns positive result', () => {
        const target = new Division(5, 2);
        expect(target.result()).to.equal(2.5);
    });

    it('should divide one positive and one negative numbers. Returns negative result', () => {
        const target = new Division(5, -2);
        expect(target.result()).to.equal(-2.5);
    });

    it('should divide two negative numbers. Returns positive result', () => {
        const target = new Division(-5, -2);
        expect(target.result()).to.equal(2.5);
    });

    it('should return cirtuclation fraction', () => {
        const target = new Division(1, 3);
        expect(target.result()).to.equal(0.3333333333333333);
    });
});
```

В данном академическом примере мы проверяем тоже важные кейсы, однако один пропустили - кейс деления на ноль. Добавим же его:

```typescript
// tests/division.test.ts
import { expect } from 'chai';
import { Division } from '../src/division';

describe('Division', () => {
    // ...
    // ...
    // ...

    it('should catch error if we divide by zero', () => {
        const target = new Division(1, 0);
        expect(() => target.result()).to.throw();
    });
});
```

## 13. Крикун (Loudmouth)

_Без логов никуда! Код сделал то, что нужно? Пиши в лог об этом. Код зашел в блок if()? Отлично, пиши в лог. А если код зашел в ... else {} ...? Об этом тоже нужно сообщить. Логов много не бывает!_

![Bow cried wolf, 1687](/images/blog/development/2023-03-04/Boycriedwolfbarlow.jpg)

Нет, бывает. Тест-крикун выдает слишком много информации, полезной разработчику во время отладки, но точно не во время прогона CI/CD пайплайна. Избавляйтесь от лишних информационных записей, сообщающих лишь то, что все идет по плану - пишите в консоль тогда, когда что-то пошло не так. В обилии информации разработчик может пропустить важную информацию, которая поможет ему понять, что произошло.

```typescript
// src/division.ts
export class Division {
    constructor(
    private readonly a: number,
    private readonly b: number) {}

    result(): number {

        if (this.a > 0) console.log('a is positive 0');
        if (this.a < 0) console.log('a is negative');

        if (this.b > 0) console.log('b is positive');
        if (this.b < 0) console.log('b is negative');

        if (this.b === 0) {
            console.log('b equals to zero, division by zero error');
            throw new Error('Division by zero!');
        }

        return this.a / this.b;
    }
}
```

Консоль при выполнении тестов класса выше:

```bash
Division
    a is positive 0
    b is positive
    ✔ should divide positive numbers. Returns positive result

    a is positive 0
    b is negative
    ✔ should divide one positive and one negative numbers. Returns negative result

    a is negative
    b is negative
    ✔ should divide two negative numbers. Returns positive result

    a is positive 0
    b is positive
    ✔ should return cirtuclation fraction

    a is positive 0
    b equals to zero, division by zero error
    ✔ should catch error if we divide by zero

```

## 14. Жадина (Greedy Catcher)

_Пишешь тест, проверяющий выпадение ошибки? Просто отлавливай исключения и все. Какая разница, какая ошибка возникла? Она ведь возникла же, а значит тест надежный!_

![Krabs](/images/blog/development/2023-03-04/Krabs_artwork.png)

Ошибка-то возникла, но где уверенность, что именно та, которую мы ожидаем при определенных условиях? "Жадиной" можно назвать тест, который отлавливают ошибку, но не проверяет причину ее причину. В примере ниже мы валидируем email, но не проверяем, что именно за ошибка возникла - невалидный email или пустое значение пришло в конструктор.

```typescript
// src/email.ts
export class Email {
    private static emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;

    private readonly email: string;

    constructor(email: string | null) {
        if (email == null) {
            throw new Error('Email cannot be null');
        }

        this.email = email;
    }

    get(): string {
        if (Email.emailRegex.test(this.email)) {
            return this.email;
        }

        throw new Error('Invalid email');
    }
}
```

```typescript
// test/email.test.ts
import { expect } from 'chai';
import { Email } from '../src/email';

describe('Email', () => {
  it('should return email for valid cases', () => {
    const validEmails = [
      'test@example.com',
      'jane.doe@example.co.uk',
      'john_doe@example.io',
    ];

    validEmails.forEach(email => {
      expect(new Email(email).get()).to.equal(email);
    });
  });

  it('should throw error for invalid emails', () => {
    const invalidEmails = [
      'test@example',
      'test@.com',
      'test@.com.',
      'test@.com.',
      '@example.com',
      null,
      ''
    ];

    invalidEmails.forEach(email => {
      expect(() => new Email(email).get()).to.throw();
    });
  });
});
```

Нужно проверять не только факт наличия ошибки, но и удостовериться, что она произошла по ожидаемой причине. В строго типизированных языках программирования можно проверить тип класса эксепшена, в JS - ассертить текст сообщения.

```typescript
// test/email.test.ts
import { expect } from 'chai';
import { Email } from '../src/email';

describe('Email', () => {
    it('should throw error for null or empty string', () => {
        const invalidEmails = [ null, '' ];

        invalidEmails.forEach(email => {
            expect(() => new Email(email)).to.throw('Email cannot be null');
        });
    });

    it('should throw error for invalid emails', () => {
        const invalidEmails = [
        'test@example',
        'test@.com',
        'test@.com.',
        'test@.com.',
        '@example.com'
        ];

        invalidEmails.forEach(email => {
            expect(() => new Email(email).get()).to.throw('Invalid email');
        });
    });
});
```

## 15. Enumerator (aka Test With No Name)

_Не трать свой креатив на имена тестов, все равно их никто не читает._

![New folder (2)](/images/blog/development/2023-03-04/new_folder_1.png)

Когда тесткейсов много, а релиз близко, велик соблазн сэкономить время на придумании правильных названий для тестов. В итоге имеем подобное:

```typescript
import { expect } from 'chai';

describe('Some logic', () => {
    it('should return true for valid case 1', () => {
        // some arrange for test by the first valid conditions
        // some act
        // some assert
        expect(true).to.equal(true);
    });

    it('should return true for valid case 2', () => {
        // some arrange for test by the second valid conditions
        // some act
        // some assert
        expect(true).to.equal(true);
    });

    it('should return true for valid case N', () => {
        // some arrange for test by the Nth valid conditions
        // some act
        // some assert
        expect(true).to.equal(true);
    });
});
```

Правильно подобранные имена позволят в будущем быстрее разбираться, почему после рефакторинга тест упал и что нужно исправлять: отрефакторенный код или тесткейс, который не соответствует новым требованиям и условиям.

## 16. Free Ride (aka Piggyback)

_Сделал багфикс, отправил на МР, а тимлид говорит, что нужно добавить юниттест? Ну и ладно, не парься, есть же уже написанные тесты. Добавь просто еще один ассерт в существующий тест. Это же легче, чем новый тест-кейс писать._

![Star wars](/images/blog/development/2023-03-04/star_wars.png)

Часто бывает, что класс поменяли незначительно, но новый тест придумывать и писать лень. Поэтому вместо нового тесткейса мы добавляем дополнительный ассерт в один из написанных. Представим ситуацию: у нас есть класс User и тест к нему:

```typescript
// src/user.ts
export class User {
    constructor(
    public readonly email: string,
    public readonly firstName: string,
    public readonly lastName: string,
    private isEmailVerified = false) {}

    verify(): void {
        if (this.isEmailVerified) {
            throw new Error('Email already verified');
        }

        this.isEmailVerified = true;
    }

    isVerified(): boolean {
        return this.isEmailVerified;
    }
}
```

```typescript
// tests/user.test.ts
import { expect } from 'chai';
import { User } from '../src/user';

describe('User', () => {
    it('should throw an error if we try to verify verified user', () => {
        const target = new User(
        'john.doe@gmail.com',
        'John',
        'Doe',
        true);

        expect(() => target.verify()).to.throw('Email already verified');
    });
});
```

Теперь мы добавляем новый метод isSocialEmail:

```typescript
// ....
isSocialEmail(): boolean {
    return this.email.endsWith('@facebook.com') ||
    this.email.endsWith('@gmail.com');
}
// ....
```

Теперь дополним тесткейс:

```typescript
it('should throw an error if we try to verify verified user', () => {
    const target = new User(
    'john.doe@gmail.com',
    'John',
    'Doe',
    true);

    expect(() => target.verify()).to.throw('Email already verified');
    expect(target.isSocialEmail()).to.be.true;
});
```

Лучше всего будет добавить новый тесткейс, чтобы тесты были независимые и атомарные. В случае, если логика метода verify() сломается и тест будет красным, до проверки isSocialEmail() выполнение даже не дойдет. Получается, что тесткейс на новый функционал не независим и игнорируется в данном случае.

## 17. Excessive Setup (aka Mother Hen)

_Настройки, настройки, сетап и еще настройки! Ну и что, что тест-кейс требует миллионы строк настройки условий? Не разделяй код, не рефактори, оставь как есть и бери следующую задачу._

![Homer likes setups](/images/blog/development/2023-03-04/homer.jpeg)

Тесты, которые требуют много кода для настройки условий, тяжело поддерживать. Иногда фича настолько объемная, что ей нужен такой объем предусловий, но все же это повод задуматься, а не сильно ли много ответственности у класса? А может лучше разделить?

![Long arrange, short assertion](/images/blog/development/2023-03-04/long_arrange_short_assert.png)

В таком тесте тяжело понять суть проверки. Кажется, что настройка слишком "шумная" и сбивает с толку. Если встретили в своем рабочем проекте подобное, то задумайтесь, не повод ли это отрефакторить класс.

## 18. Line hitter

_Ты тимлид и ты усвоил урок, что стопроцентное покрытие кода - это зло. Ну что ж, в публичных методах мы же можем обеспечить полное покрытие, не так ли?_

![John Wick](/images/blog/development/2023-03-04/john_wick.jpeg)

На первый взгляд, тест в примере ниже покрывает 100% кода метода, однако суть работы особо и не проверяется и выходные данные не анализируются.

```typescript
// src/calculator.ts
export class Calculator {
    constructor(
    private readonly first: number){}

    add(second: number): number {
        return this.first + second;
    }

    subtract(second: number): number {
        return this.first - second;
    }

    multiply(second: number): number {
        return this.first * second;
    }

    divide(second: number): number {
        return this.first / second;
    }
}
```

```typescript
// test/calculator.test.ts
import { expect } from 'chai';
import { Calculator } from '../src/calculator';

describe('Calculator', () => {
    it('should call methods', () => {
        const add = new Calculator(1).add(1);
        const substract = new Calculator(1).subtract(1);
        const multiply = new Calculator(1).multiply(1);
        expect(new Calculator(1).divide(1)).to.equal(1);

    });
});
```

Снова в погоне за метрикой мы упускаем суть тестирования. Метрика - это хорошо, но не самоцель. Сначала напиши тест-кейсы на каждый сценарий, покрой критичный функционал и придуманные пограничные кейсы, а уже потом посмотри на покрытие кода. Если какой-то код остался непокрытым, то может это код нужно удалить, а не тест-кейс добавить?

## 19. Лжец (aka [Evergreen Tests](https://youtu.be/1Z_h55jMe-M?t=1059), Success Against All Odds)

_Поменял логику работы метода, но тесты не упали? Отлично, какой ты молодец, что сумел написать такие надежные тесты. Так держать!_

![Liar](/images/blog/development/2023-03-04/liar.png)

"Лжецом" называют тест, который будет зеленый, даже если код тестируемого метода поменяли. Для демонстрации можно вспомнить один из прошлых примеров:

```typescript
// src/user.ts
export class User {

    private deletedAt: Date | null = null

    constructor(
    public readonly email: string,
    private isEmailVerified = false,
    ) {}

    verify(): void {
        if (this.isEmailVerified) {
            throw new Error('Email already verified');
        }

        this.isEmailVerified = true;
    }

    isVerified(): boolean {
        return !this.isDeleted() && this.isEmailVerified;
    }

    isSocialEmail(): boolean {
        return this.email.endsWith('@facebook.com') ||
        this.email.endsWith('@gmail.com');
    }

    delete(): void {
        if (this.deletedAt != null) {
            throw new Error('User is already removed');
        }

        this.deletedAt = new Date();
    }

    isDeleted(): boolean {
        return this.deletedAt != null;
    }
}
```

```typescript
// tests/user.test.ts
import { expect } from 'chai';
import { User } from '../src/user';

describe('User', () => {
    it('.isVerified should return false if user was removed', () => {
        const target = new User('john.doe@gmail.com', false);

        target.delete();
        expect(target.isVerified()).to.be.false;
    });
});
```

Теперь поменяем проверку в методе isVerified():

```typescript
isVerified(): boolean {
    return /*!this.isDeleted() &&*/ this.isEmailVerified;
}
```

Тест до сих пор зеленый даже при условии, что код метода был изменен:

```typescript
it('.isVerified should return false if user was removed', () => {
    const target = new User('john.doe@gmail.com', false);

    target.delete();
    expect(target.isVerified()).to.be.false;
});
```

Все потому, что тест не проверяет подробно состояние тестируемого объекта и не удостоверяется, какие именно условия привели к ожидаемому результату. В качестве решения можно добавить мутационные тесты, которые будут проверять, что тесты не лгут. Мутационный тест - это тест, который меняет или код бизнес-классов, или ассерты в тестах, и проверяет, что тесты упали. Если тесты не упали, то это значит, что доверия к ним нет.

## Заключение

В этой статье мы рассмотрели 19 антипаттернов в юнит-тестировании. Надеюсь, что теперь ты попадешь в эти ловушки. Код юниттестов - не second-class-citizen, их тоже нужно писать "чистыми" и понятными. Практикуйся чаще, и тогда код твоего проекта будет надежней, тестировщик будет меньше на тебя ругаться, а тимлид - меньше краснеть.

## Источники

- https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue
- https://archive.is/3acB#selection-119.0-119.17
- https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html
- https://blog.codepipes.com/testing/software-testing-antipatterns.html
