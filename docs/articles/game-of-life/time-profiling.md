# Профилирование по времени

В этой главе мы улучшим производительность нашей реализации «Игры жизни». Будем опираться на замеры времени (time profiling), чтобы не действовать вслепую.

Перед продолжением ознакомьтесь с [доступными инструментами для профилирования по времени кода на Rust и WebAssembly](https://rustwasm.github.io/docs/book/reference/time-profiling.html#available-tools).

## Счётчик FPS на основе `window.performance.now`

Такой счётчик кадров в секунду пригодится, когда будем разбираться, как ускорить отрисовку «Игры жизни».

Начнём с добавления объекта `fps` в `wasm-game-of-life/www/index.js`:

```js
const fps = new (class {
    constructor() {
        this.fps = document.getElementById('fps');
        this.frames = [];
        this.lastFrameTimeStamp = performance.now();
    }

    render() {
        // Переводим интервал с предыдущего кадра в оценку кадров в секунду.
        const now = performance.now();
        const delta = now - this.lastFrameTimeStamp;
        this.lastFrameTimeStamp = now;
        const fps = (1 / delta) * 1000;

        // Храним только последние 100 измерений.
        this.frames.push(fps);
        if (this.frames.length > 100) {
            this.frames.shift();
        }

        // Ищем минимум, максимум и среднее по последним 100 значениям.
        let min = Infinity;
        let max = -Infinity;
        let sum = 0;
        for (let i = 0; i < this.frames.length; i++) {
            sum += this.frames[i];
            min = Math.min(this.frames[i], min);
            max = Math.max(this.frames[i], max);
        }
        let mean = sum / this.frames.length;

        // Выводим статистику.
        this.fps.textContent = `
Кадров в секунду:
         последний = ${Math.round(fps)}
 среднее за 100 = ${Math.round(mean)}
    мин за 100 = ${Math.round(min)}
    макс за 100 = ${Math.round(max)}
`.trim();
    }
})();
```

Далее вызываем `fps.render()` на каждой итерации `renderLoop`:

```js
const renderLoop = () => {
    fps.render(); // новое

    universe.tick();
    drawGrid();
    drawCells();

    animationId = requestAnimationFrame(renderLoop);
};
```

Не забудьте добавить элемент `fps` в `wasm-game-of-life/www/index.html` чуть выше `<canvas>`:

```html
<div id="fps"></div>
```

И CSS, чтобы вывод выглядел аккуратно:

```css
#fps {
    white-space: pre;
    font-family: monospace;
}
```

Готово! Обновите [http://localhost:8080](http://localhost:8080) — на странице появится счётчик FPS.

### Замер каждого `Universe::tick` через `console.time` и `console.timeEnd`

Чтобы измерить длительность каждого вызова `Universe::tick`, можно использовать `console.time` и `console.timeEnd` из крейта `web-sys`.

Сначала добавьте `web-sys` в зависимости `wasm-game-of-life/Cargo.toml`:

```toml
[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

На каждый вызов `console.time` должен приходиться соответствующий `console.timeEnd`, поэтому удобно обернуть оба вызова в тип с паттерном [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization):

```rust
extern crate web_sys;
use web_sys::console;

pub struct Timer<'a> {
    name: &'a str,
}

impl<'a> Timer<'a> {
    pub fn new(name: &'a str) -> Timer<'a> {
        console::time_with_label(name);
        Timer { name }
    }
}

impl<'a> Drop for Timer<'a> {
    fn drop(&mut self) {
        console::time_end_with_label(self.name);
    }
}
```

Затем можно замерить `Universe::tick`, добавив в начало метода:

```rust
let _timer = Timer::new("Universe::tick");
```

Длительность каждого вызова `Universe::tick` теперь попадает в консоль:

[Снимок экрана: логи console.time](./console-time.png)

Кроме того, пары `console.time` / `console.timeEnd` отображаются во «временной шкале» или «водопаде» в профилировщике браузера:

[Снимок экрана: console.time в профилировщике](./console-time-in-profiler.png)

## Увеличиваем вселенную «Игры жизни»

> ⚠️ В этом разделе для примеров используются скриншоты из Firefox. В других браузерах инструменты похожи, но названия и вид панелей могут чуть отличаться. Суть профиля будет той же, а вот удобство отдельных представлений может различаться.

Что будет, если сделать вселенную больше? Если заменить сетку 64×64 на 128×128 (изменив `Universe::new` в `wasm-game-of-life/src/lib.rs`), на моей машине FPS падает с ровных 60 до неровных ~40.

Если записать профиль и открыть вид «водопада», видно, что кадр анимации занимает больше 20 миллисекунд. При 60 FPS на весь кадр остаётся около шестнадцати миллисекунд — и это не только наш JavaScript и WebAssembly, но и остальная работа браузера, в том числе отрисовка.

[Снимок экрана: водопад отрисовки кадра](./drawCells-before-waterfall.png)

Если разобрать один кадр анимации, оказывается, что сеттер `CanvasRenderingContext2D.fillStyle` очень дорогой!

> ⚠️ В Firefox, если вместо `CanvasRenderingContext2D.fillStyle` вы видите строку просто «DOM», включите в настройках инструментов производительности опцию «Show Gecko Platform Data»: [Включение Show Gecko Platform Data](./profiler-firefox-show-gecko-platform.png)

[Снимок экрана: flame graph отрисовки кадра](./drawCells-before-flamegraph.png)

На агрегированном дереве вызовов по многим кадрам видно, что это не случайность:

[Снимок экрана: call tree отрисовки кадра](./drawCells-before-calltree.png)

Почти 40% времени уходит на этот сеттер!

> ⚡ Можно было ожидать, что узким местом окажется что-то в методе `tick`, но это не так. Всегда опирайтесь на профилирование: время часто тратится не там, где вы его ждёте.

В функции `drawCells` в `wasm-game-of-life/www/index.js` свойство `fillStyle` задаётся для каждой клетки вселенной на каждом кадре анимации:

```js
for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col);

        ctx.fillStyle =
            cells[idx] === DEAD ? DEAD_COLOR : ALIVE_COLOR;

        ctx.fillRect(
            col * (CELL_SIZE + 1) + 1,
            row * (CELL_SIZE + 1) + 1,
            CELL_SIZE,
            CELL_SIZE
        );
    }
}
```

Раз смена `fillStyle` так дорога, как реже её вызывать? Значение должно зависеть от того, жива клетка или мёртва. Если выставить `fillStyle = ALIVE_COLOR` и в одном проходе нарисовать все живые клетки, затем `fillStyle = DEAD_COLOR` и во втором проходе — все мёртвые, `fillStyle` меняется только дважды, а не на каждую клетку.

```js
// Живые клетки.
ctx.fillStyle = ALIVE_COLOR;
for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col);
        if (cells[idx] !== Cell.Alive) {
            continue;
        }

        ctx.fillRect(
            col * (CELL_SIZE + 1) + 1,
            row * (CELL_SIZE + 1) + 1,
            CELL_SIZE,
            CELL_SIZE
        );
    }
}

// Мёртвые клетки.
ctx.fillStyle = DEAD_COLOR;
for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
        const idx = getIndex(row, col);
        if (cells[idx] !== Cell.Dead) {
            continue;
        }

        ctx.fillRect(
            col * (CELL_SIZE + 1) + 1,
            row * (CELL_SIZE + 1) + 1,
            CELL_SIZE,
            CELL_SIZE
        );
    }
}
```

После сохранения и обновления [http://localhost:8080/](http://localhost:8080/) отрисовка снова держит ровные 60 FPS.

По новому профилю видно, что кадр теперь занимает около десяти миллисекунд.

[Снимок экрана: водопад после изменений в drawCells](./drawCells-after-waterfall.png)

В разборе одного кадра стоимость `fillStyle` исчезла, большая часть времени уходит на `fillRect` — прямоугольники клеток.

[Снимок экрана: flame graph после изменений в drawCells](./drawCells-after-flamegraph.png)

## Больше тиков за кадр

Кому-то не хочется ждать: хочется не один тик вселенной за кадр анимации, а девять. Функцию `renderLoop` в `wasm-game-of-life/www/index.js` можно изменить так:

```js
for (let i = 0; i < 9; i++) {
    universe.tick();
}
```

На моей машине FPS снова падает примерно до 35 — плохо, хочется плавные 60.

Теперь ясно, что время уходит в `Universe::tick`: добавим несколько `Timer` с вызовами `console.time` / `console.timeEnd` в разных местах и посмотрим. Гипотеза: выделение нового вектора клеток и освобождение старого на каждом тике дороги и съедают заметную долю бюджета.

```rust
pub fn tick(&mut self) {
    let _timer = Timer::new("Universe::tick");

    let mut next = {
        let _timer = Timer::new("allocate next cells");
        self.cells.clone()
    };

    {
        let _timer = Timer::new("new generation");
        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);

                let next_cell = match (cell, live_neighbors) {
                    // Правило 1: живая клетка с менее чем двумя соседями умирает (недонаселение).
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // Правило 2: живая клетка с двумя или тремя соседями выживает.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // Правило 3: живая клетка с более чем тремя соседями умирает (перенаселение).
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // Правило 4: мёртвая клетка с ровно тремя соседями оживает.
                    (Cell::Dead, 3) => Cell::Alive,
                    // Остальные клетки не меняются.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }
    }

    let _timer = Timer::new("free old cells");
    self.cells = next;
}
```

По замерам видно, что гипотеза неверна: подавляющее время уходит на вычисление следующего поколения клеток. Выделение и освобождение вектора на каждом тике, к удивлению, почти ничего не стоят. Ещё один повод всегда сверяться с профилем.

[Снимок экрана: таймеры внутри Universe::tick](./console-time-in-universe-tick.png)

Для следующего фрагмента нужен компилятор `nightly` из‑за [встроенного бенчмарка](https://doc.rust-lang.org/unstable-book/library-features/test.html). Ещё понадобится утилита [cargo benchcmp](https://github.com/BurntSushi/cargo-benchcmp) — небольшой инструмент для сравнения микробенчмарков из `cargo bench`.

Напишем нативный `#[bench]`, делающий то же, что и WebAssembly, но уже со зрелыми средствами профилирования. Новый файл `wasm-game-of-life/benches/bench.rs`:

```rust
#![feature(test)]

extern crate test;
extern crate wasm_game_of_life;

#[bench]
fn universe_ticks(b: &mut test::Bencher) {
    let mut universe = wasm_game_of_life::Universe::new();

    b.iter(|| {
        universe.tick();
    });
}
```

Также придётся закомментировать все аннотации `#[wasm_bindgen]` и секцию `"cdylib"` в `Cargo.toml`, иначе сборка нативного кода завершится ошибками линковки.

После этого выполните `cargo bench | tee before.txt`, чтобы скомпилировать и прогнать бенчмарк. Часть `| tee before.txt` перенаправит вывод `cargo bench` в файл `before.txt`.

```
$ cargo bench | tee before.txt
    Finished release [optimized + debuginfo] target(s) in 0.0 secs
     Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/release/deps/bench-8474091a05cfa2d9

running 1 test
test universe_ticks ... bench:     664,421 ns/iter (+/- 51,926)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out
```

Из вывода видно, где лежит бинарник; тот же бенчмарк можно запустить под профилировщиком ОС. У меня Linux, поэтому использую [`perf`](https://perf.wiki.kernel.org/index.php/Main_Page):

```
$ perf record -g target/release/deps/bench-8474091a05cfa2d9 --bench
running 1 test
test universe_ticks ... bench:     635,061 ns/iter (+/- 38,764)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out

[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.178 MB perf.data (2349 samples) ]
```

`perf report` показывает, что время, как и ожидалось, уходит в `Universe::tick`:

[Снимок экрана: отчёт perf](./bench-perf-report.png)

В `perf` можно разметить инструкции по доле времени, нажав `a`:

[Снимок экрана: аннотация инструкций в perf](./bench-perf-annotate.png)

Так видно, что 26,67% времени уходит на суммирование значений соседних клеток, 23,41% — на получение индекса столбца соседа и ещё 15,42% — на индекс строки. Среди трёх самых дорогих инструкций две последние — это `div`. Эти деления реализуют логику индексации по модулю в `Universe::live_neighbor_count`.

Напомним определение `live_neighbor_count` в `wasm-game-of-life/src/lib.rs`:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;
    for delta_row in [self.height - 1, 0, 1].iter().cloned() {
        for delta_col in [self.width - 1, 0, 1].iter().cloned() {
            if delta_row == 0 && delta_col == 0 {
                continue;
            }

            let neighbor_row = (row + delta_row) % self.height;
            let neighbor_col = (column + delta_col) % self.width;
            let idx = self.get_index(neighbor_row, neighbor_col);
            count += self.cells[idx] as u8;
        }
    }
    count
}
```

Модulo выбрали, чтобы не засорять код ветвлениями для первой/последней строки и столбца. Но цена инструкции `div` платится и в самом частом случае, когда ни строка, ни столбец не на границе и «заворот» по модулю не нужен. Если для краёв использовать `if` и развернуть цикл, ветки _должны_ хорошо предсказываться блоком предсказания ветлений в CPU.

Перепишем `live_neighbor_count` так:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;

    let north = if row == 0 {
        self.height - 1
    } else {
        row - 1
    };

    let south = if row == self.height - 1 {
        0
    } else {
        row + 1
    };

    let west = if column == 0 {
        self.width - 1
    } else {
        column - 1
    };

    let east = if column == self.width - 1 {
        0
    } else {
        column + 1
    };

    let nw = self.get_index(north, west);
    count += self.cells[nw] as u8;

    let n = self.get_index(north, column);
    count += self.cells[n] as u8;

    let ne = self.get_index(north, east);
    count += self.cells[ne] as u8;

    let w = self.get_index(row, west);
    count += self.cells[w] as u8;

    let e = self.get_index(row, east);
    count += self.cells[e] as u8;

    let sw = self.get_index(south, west);
    count += self.cells[sw] as u8;

    let s = self.get_index(south, column);
    count += self.cells[s] as u8;

    let se = self.get_index(south, east);
    count += self.cells[se] as u8;

    count
}
```

Снова запустите бенчмарк, на этот раз сохранив вывод в `after.txt`.

```
$ cargo bench | tee after.txt
   Compiling wasm_game_of_life v0.1.0 (file:///home/fitzgen/wasm_game_of_life)
    Finished release [optimized + debuginfo] target(s) in 0.82 secs
     Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/release/deps/bench-8474091a05cfa2d9

running 1 test
test universe_ticks ... bench:      87,258 ns/iter (+/- 14,632)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out
```

Намного лучше! Насколько именно — покажет `benchcmp` и два сохранённых файла:

```
$ cargo benchcmp before.txt after.txt
 name            before.txt ns/iter  after.txt ns/iter  diff ns/iter   diff %  speedup
 universe_ticks  664,421             87,258                 -577,163  -86.87%   x 7.61
```

Ускорение примерно в 7,61 раза.

WebAssembly сознательно близок к распространённым архитектурам, но стоит убедиться, что выигрыш нативного кода переносится и на `.wasm`.

Пересоберите модуль командой `wasm-pack build` и обновите [http://localhost:8080/](http://localhost:8080/). У меня снова стабильные 60 FPS, а профиль браузера показывает около десяти миллисекунд на кадр.

Готово!

[Снимок экрана: водопад после замены modulo на ветвления](./waterfall-after-branches-and-unrolling.png)

## Упражнения

-   Следующий очевидный шаг для ускорения `Universe::tick` — убрать выделение и освобождение памяти на каждом тике. Реализуйте двойную буферизацию: у `Universe` два вектора, ни один не освобождается, в `tick` новые буферы не выделяются.

-   Реализуйте дельта-подход из главы «Реализация Life», когда Rust возвращает в JavaScript список изменившихся клеток. Ускоряет ли это отрисовку на `<canvas>`? Получится ли обойтись без нового списка дельт на каждом тике?

-   Профилирование показало, что отрисовка на 2D `<canvas>` не самая быстрая. Замените рендер на [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API). Насколько быстрее вариант на WebGL? До какого размера вселенной узким местом остаётся именно WebGL?
