---
layout: post
title: Антипаттерны тестирования программ с примерами
category: development
tags: [.net, elegant_objects]
date: "2023-03-04"
description: ""
draft: true
---

## 0. Странник (Stranger)

Тест, который хоть и тестирует методы одного класса, но по факту проверяет функционал другого класса, находящегося внутри тестируемого.

Представим ситуацию, когда у нас есть один бизнес-сервис и другой, который используется в первом в качестве зависимости:

```typescript
// src/another-business-service.ts
export class AnotherBusinessService {
    ReturnRandomNumber(): number {
        return 42;
    }
}
```

```typescript
// src/business-service.ts
import { AnotherBusinessService } from './another-business-service';

export class BusinessService {

    constructor(
    private readonly anotherBusinessService: AnotherBusinessService
    ) {}

    ReturnRandomNumberAsString(): string {
        return this.anotherBusinessService.ReturnRandomNumber().toString();
    }
}
```

Теперь тест:

```typescript
// tests/business-service.test.ts
import { AnotherBusinessService } from './another-business-service';
import { BusinessService } from './business-service';

describe('BusinessService', () => {
    it('ReturnRandomNumber returns 42', () => {
        const target = new BusinessService(
        new AnotherBusinessService());

        const result = target.ReturnRandomNumberAsString();
        expect(result).toBe('42');
    });
});
```

Несмотря на то, что внешне мы тестируем `BusinessLogic`, на самом деле наши тест кейсы проверяют логику `AnotherBusinessService`. Если еще вариация этого антипаттерна - ["Forty-Foot Pole"](https://stackoverflow.com/a/339247). Так называют тесты, проверяющие функционал тех классов, которые лежат не за одним слоем абстракции глубоко внутри.

## 1. Один тест на каждый метод (test-per-method)

Иметь по одному тесту на каждый метод рабочего кода - это хорошее начало, но так функционал системы и пограничные кейсы не проверишь. Останавливаться на этом не стоит.

Тестируемый класс:

```typescript
// src/testable-class.ts
export class TestableClass {
    MakeSomeLogic1(): number {
        return 42;
    }

    MakeSomeLogic2(): number {
        return 43;
    }

    MakeSomeLogic3(): number {
        return 44;
    }
}
```

и тесты:

```typescript
// tests/testable-class.test.ts
import { TestableClass } from './anal-probe-example';

describe('TestableClass', () => {
    it('MakeSomeLogic1 returns 42', () => {
        const target = new TestableClass();
        const result = target.MakeSomeLogic1();
        expect(result).toBe(42);
    });

    it('MakeSomeLogic2 returns 43', () => {
        const target = new TestableClass();
        const result = target.MakeSomeLogic2();
        expect(result).toBe(43);
    });

    it('MakeSomeLogic3 returns 44', () => {
        const target = new TestableClass();
        const result = target.MakeSomeLogic3();
        expect(result).toBe(44);
    });
});
```

 В примере код возвращает константы, но обычно такое бывает редко. Нужно проверять дальше пограничные кейсы и ошибочные сценарии.

## 2. Анальная пробка (Anal probe)

Тестирование приватных полей, методов. Тест знает слишком много о тестируемом классе и лезет не в свое дело. Покажу на примере. Тут у нас тестируемый класс, который изменяет приватное поле:

```typescript
// src/testable-class.ts
export class TestableClass {
    constructor(private value: string) {}

    public MakeSomeLogic(): void {
        this.value += ', 1';
    }
}
```

Тут у нас тест, который проверяет значение приватного поля:

```typescript
// tests/testable-class.test.ts
import { TestableClass } from './anal-probe-example';

describe('TestableClass', () => {
    it('value should be changed', () => {
        const target = new TestableClass('example');

        target.MakeSomeLogic();
        expect((target as any).value).toBe('example, 1');
    });
});
```

В случае, если приватное поле переименуют или изменят логику так, что приватное поле не понадобится, тест будет падать. Проверять нужно выходные публичные данные или хранилище, которым манипулирует тестируемый класс.

## 3. Happy path

Тесты проверяют только прохождение "частливого пути", не затрагивая пограничные кейсы и ошибочные сценарии. Например, у нас есть класс проверки возраста:

```typescript
// src/age-detector.ts
export class AgeDetector {

    constructor(
    private readonly yearOfBirth: number
    ) {}

    howOldAmI(): number {
        const currentYear = new Date().getFullYear();
        return currentYear - this.yearOfBirth;
    }
}
```

И тест-класс:

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

Проверили, что возвращает разницу. Но что насчет кейсов, когда дата рождения еще не наступила? А если передали дату в будущем в конструктор класса? А если передали отрицательное число? И таких пограничных кейсов может быть тысячи, поэтому не стоит останавливаться в написании тестов на только лишь одном успешном сценарии.

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

## 4. Слоупок (Slowpoke)

Тесты, которые воспроизводятся очень долго. Например, после старта тестов можно смело пойти заварить себе чай или пойти в курилку. Тесты могут быть медленными, если они работают с внешними системами, делают запросы в другие сервисы или взаимодействуют с файловой системой. Такие тесты ненадежны и по другим причинам, но в данном кейсе они еще и увеличивают время прогона пайплайнов. Нужно мокать, подменять сервисы заглушками и рефакторить, чтобы в тестах было как можно меньше взаимодействия со сторонними системами.

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

Если тестов, которые зависят от внешних факторов, много, то и время исполнения будет рандомным и зависеть от состояния нагруженности сети, зависеть от погоды, etc. Старайтесь мокать вызовы внешних систем.

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

Тестфайл содержит в себе тысячи строк кода. Такое может быть, когда тестируемый класс содержит настолько много логики и ответственности, что одной тысячей строк кода не обойтись. Что делать в этом случае? Только дробить и класс, и файл тестов соответственно. В куче строк кода легко потеряться.

![1k lines of test](/images/blog/development/2023-03-04-unit-testing-anti-patterns/1_k_line_of_tests.png)

![9k lines of logic](/images/blog/development/2023-03-04-unit-testing-anti-patterns/over9000.png)

## 6. Mockery

Моки - это отличный способ подменить функциональность. Так можно протестировать свою бизнес-логику, которая зависит на ответе стороннего сервиса, и при эётом не вызывать этот сторонний сервис. Но и перебарщивать с этим не стоит, иначе можно столкнуться с ситуацией, когда вы мокаете сервисы, а потом проверяете работу этих же моков. Нужно соблюдать баланс.

Например, на бэкенде популярен паттерн "репозиторий" для создания прослойки между бизнес-логикой и хранилищем данных. Ниже можно увидеть, как мы настраиваем с помощью моков класс, но внутри класса логики-то толком и нет

![Setup mocks for testclass](/images/blog/development/2023-03-04-unit-testing-anti-patterns/mock_dotnet.png)

А ниже - сам класс. Логики внутри почти нет, в итоге тестировать и нечего особо.

![Setup mocks for testclass](/images/blog/development/2023-03-04-unit-testing-anti-patterns/class_under_tests_dotnet.png)

Даже такие классы нужно тестировать, но в данном случае лучше подойдет мок всей БД, куда сначала мы кладем искомый объект, а потом уже проверяем, что он будет возвращен бизнес-логикой.

## 7. Инспектор (Inspector)

В погоне за метрикой покрытия тестами можно так увлечься, что люди теряют смысл этой метрики. Вследствие этого появляются тесты, которые проверяют настолько глубокий функционал внутри класса, что при любом рефакторинге нужно переписывать и тест. Этот антипаттерн схож с "анальной пробкой", но тут иная цель: если анальная пробка преследует цель проверить результат работы логики по скрытым полям класса, то тут целью ставят полное покрытие кода тестами, а качество тестов значения не имеет.

Есть открытый [вопрос на SO](https://stackoverflow.com/q/4520216), заданный Егором Бугаенко, о том, как написать тест для приватного конструктора класса в Java. Вопрос был задан скорее для того, чтобы собрать мнение сообщества по вопросу, и [один из лучших ответов](https://stackoverflow.com/a/4520241) я приведу тут:

> Remember that coverage is meant to be something which is useful to you - you should be in charge of the tool, not the other way round.

## 8. Generous Leftovers (aka Chain Gang, Wet Floor).

Этот антипаттерн описывает ситуацию, когда один тест создает некоторые данные, нужные для другого теста, и записывает их в некое хранилище. Затем второй тест читает данные и проверяет некоторую логику. Например, такой подход могут использовать для проверки логики обработки репортов.

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

Подход плохой, потому что:

- Возникает необходимость запускать некоторые тесты в строгом порядке
- Возникает зависимость тестов друг от друга, они не атомарны

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

## 9. Local Hero (aka "Скрытая зависимость", "Евангелист ОС", "Хулиган окружения").

Так называют код или тест, который зависит от переменных окружения. В зависимости от окружения, тест или выполняется, или фейлится. Независимым такой тест назвать сложно. Например, какой-то бизгнес-класс зависит от сервера:

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

## 10. Придира (Nitpicker)

Так называют тесты, которые проверяю лишь незначительную часть выходных данных тестируемого метода. Например, клас формирует репорт большого размера, и мы проверяем весь выходной текст на соответствие ожидаемому результату. Казалось бы, такое поведение корректно, однако в случае ошибки в одной из колонок мы получим сложночитаемую ошибку в логах выполнения тестов.

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

Лучшим выхожом будет отлавливать эксепшн и проверять его поля на ожидаемый текст ошибки, например, или иные ее поля:

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

Так можно назвать тест, который проверяет маловероятные кейсы, не оказывающие значительное влияние на поведение класса, но при этом как будто бы специально пропускает важные тесткейсы. Пример привести сложно, ведь тут все сильно зависит от бизнеса, для которого работает приложение - бизнес определяет, какие кейсы важные и какие - маловероятные.

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

Так можно назвать тест, который выдает слишком много информации, полезной разработчику во время отладки, но точно не во время прогона CI/CD пайплайна. Избавляйтесь от лишних информационных записей, сообщающих лишь то, что все идет по плану - пишите в консоль тогда, когда что-то пошло не так.

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

```
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

Так называют тесты, которые отлавливают ошибку, но не проверяют причину этой ошибки.

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

Так называют тесты, которые требуют много кода только лишь для настройки условий. Это лучше, чем отсутствие тесткейсов, однако это явный признак того, что тестируемый класс слишком много знает и слишком много делает, а это - повод задуматься о рефакторинге.

![Long arrange, short assertion](/images/blog/development/2023-03-04-unit-testing-anti-patterns/long_arrange_short_assert.png)

В таком тесте тяжело понять суть проверки. Кажется, что настройка слишком "шумная" и сбивает с толку. Если встретили в своем рабочем проекте подобное, то задумайтесь, не повод ли это отрефакторить класс.

## 18. Line hitter

На первый взгляд, тест покрывает 100% кода метода, однако суть работы особо и не проверяется и выходные данные не анализируются.

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

## 19. Лжец (aka [Evergreen Tests](https://youtu.be/1Z_h55jMe-M?t=1059), Success Against All Odds)

Так называют тест, который всегда зеленый, даже если код тестируемого метода поменяли. Для демонстрации можно вспомнить один из прошлых примеров:

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

Все потому, что тест не проверяет подробно состояние тестируемого объекта и не удостоверяется, какие именно условия привели к ожидаемому результату.

## Источники

- https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue
- https://archive.is/3acB#selection-119.0-119.17
- https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html
- https://blog.codepipes.com/testing/software-testing-antipatterns.html
