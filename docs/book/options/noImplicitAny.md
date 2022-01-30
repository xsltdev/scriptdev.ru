# noImplicitAny

Есть некоторые вещи, которые невозможно логически вывести, или такой вывод может привести к неожиданным ошибкам. Прекрасный пример - параметры функции. Если вы не опишите их, неясно, что должно и что не должно быть вылидным, например:

```ts
function log(someArg) {
  sendDataToServer(someArg);
}

// Какой параметр валидный, а какой нет?
log(123);
log('hello world');
```

Поэтому, если вы не опишите какой-либо параметр функции, TypeScript присваивает значение `any` и двигается дальше. Это по существу отключает проверку типов в таких случаях, чего и ожидает разработчик JavaScript. Но это может застать врасплох людей, которые хотят более высокой надёжности. Следовательно, есть опция `noImplicitAny`, которая при включении будет отмечать случаи, когда тип не может быть определен, например:

```ts
function log(someArg) { // Ошибка : someArg имеет неявный тип any
  sendDataToServer(someArg);
}
```

Конечно, вы можете продолжить и описать:

```ts
function log(someArg: number) {
  sendDataToServer(someArg);
}
```

Но если вы действительно хотите *нулевую надёжность*, вы можете *явно* пометить это как `any`:

```ts
function log(someArg: any) {
  sendDataToServer(someArg);
}
```