# Never

[Youtube: Видеоурок о типе never](https://www.youtube.com/watch?v=aldIFYWu6xc)

[Egghead: Видеоурок о типе never](https://egghead.io/lessons/typescript-use-the-never-type-to-avoid-code-with-dead-ends-using-typescript)

При разработке языка программирования используется понятие _bottom_ типа, который является **естественным** результатом _анализа потока кода_. TypeScript выполняет _анализ потока кода_ (😎), поэтому он должен достоверно представлять то, что никогда не произойдёт.

Тип `never` используется в TypeScript для обозначения этого типа _bottom_. Случаи, когда это происходит естественным путем:

-   Функция никогда ничего не вернёт (например, если в теле функции есть `while(true){}`)
-   Функция всегда выбрасывает ошибку (например, в `function foo () {throw new Error ('Не реализована')}` тип возвращаемого значения функции `foo` - `never`)

Конечно, вы можете использовать это описание и сами:

```ts
let foo: never; // Okay
```

Тем не менее, `never` _может быть присвоено только другое never_. Например:

```ts
let foo: never = 123; // Ошибка: Тип number нельзя присвоить типу never

// Okay, так как тип возвращаемого значения функции - `never`
let bar: never = (() => {
    throw new Error(
        'Поднимаю руки вверх, будто мне все равно'
    );
})();
```

Отлично. Теперь давайте перейдем к основному варианту использования :)

## Пример использования: тщательные проверки

Вы можете вызывать функции never в never-обстоятельствах.

```ts
function foo(x: string | number): boolean {
    if (typeof x === 'string') {
        return true;
    } else if (typeof x === 'number') {
        return false;
    }

    // Без типа never мы бы ошиблись:
    // - Не все пути кода возвращают значение (строгие проверки на null)
    // - Или обнаружен недостижимый код
    // Но поскольку TypeScript понимает, что функция `fail` возвращает `never`
    // Он может позволить вам вызвать её, поскольку вы могли бы использовать её
    // для безопасности выполнения / тщательных проверок.
    return fail('Нетщательный!');
}

function fail(message: string): never {
    throw new Error(message);
}
```

А поскольку `never` назначается только другому `never`, вы также можете использовать его для тщательных проверок во _время компиляции_. Это описано в разделе [_размеченные объединения_](./discriminated-unions.md).

## Путаница с `void`

Как только кто-то говорит вам, что возвращается `never`, когда функция никогда не завершается корректно и ничего не будет возвращено, вы интуитивно хотите думать об этом как о `void`. Однако `void` - это значение. "Never" - ложное утверждение в логике.

Функция, которая _ничего не возвращает_, возвращает значение `void`. Однако функция _которая никогда ничего не возвращает_ (или всегда выбрасывает ошибку), возвращает `never`. `void` - это то, что может быть присвоено (без `strictNullChecking`), но `never` _никогда_ не может быть присвоено чему-либо, кроме `never`.

## Логический вывод типа в функциях, которые никогда ничего не возвращают

Для объявлений функций TypeScript по умолчанию подразумевает `void`, как показано ниже:

```ts
// Предполагаемый тип возвращаемого значения: void
function failDeclaration(message: string) {
    throw new Error(message);
}
// Предполагаемый тип возвращаемого значения: never
const failExpression = function (message: string) {
    throw new Error(message);
};
```

Конечно, вы можете исправить это подробным описанием:

```ts
function failDeclaration(message: string): never {
    throw new Error(message);
}
```

Основная причина - обратная совместимость с реальным кодом JavaScript:

```ts
class Base {
    overrideMe() {
        throw new Error('Ты забыл переопределить меня!');
    }
}
class Derived extends Base {
    overrideMe() {
        // Код, который на самом деле возвращается сюда
    }
}
```

Если `Base.overrideMe`.

> Реальный TypeScript может преодолеть это с помощью `abstract` функций, но этот логический вывод поддерживается для совместимости.
