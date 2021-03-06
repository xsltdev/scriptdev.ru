# Функции с состоянием

Общей особенностью других языков программирования является использование ключевого слова `static` для увеличения _времени жизни_ (не _области_) переменной функции, чтобы она существовала за пределами вызовов функций. Вот пример в `C`, в котором достигается это:

```c
void called() {
    static count = 0;
    count++;
    printf("Called : %d", count);
}

int main () {
    called(); // Called : 1
    called(); // Called : 2
    return 0;
}
```

Поскольку JavaScript (или TypeScript) не имеет статических переменных для функций, вы можете добиться того же, используя различные абстракции, которые обертывают локальные переменные, например используя `class`:

```ts
const { called } = new (class {
    count = 0;
    called = () => {
        this.count++;
        console.log(`Called : ${this.count}`);
    };
})();

called(); // Called : 1
called(); // Called : 2
```

> Разработчики C++ также пытаются добиться этого, используя паттерн, который они называют `функтор` (класс, который переопределяет оператор `()`).
