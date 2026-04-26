---
description: Интерактивность в игре жизни — пауза, клик по клеткам и связь JavaScript с WebAssembly
---

# Добавление интерактивности

Продолжим разбирать связку JavaScript и WebAssembly: добавим в реализацию игры жизни интерактив. Пользователь сможет переключать клетку «жива/мёртва» по клику и ставить игру на паузу — так удобнее рисовать паттерны.

## Пауза и возобновление игры

Добавим кнопку, переключающую режим «играет / на паузе». В файле `wasm-game-of-life/www/index.html` вставьте кнопку сразу над `<canvas>`:

```html
<button id="play-pause"></button>
```

В JavaScript `wasm-game-of-life/www/index.js` сделайте следующее:

-   Храните идентификатор, который вернул последний вызов `requestAnimationFrame`, чтобы отменить анимацию через `cancelAnimationFrame` с этим идентификатором.

-   По клику на «play/pause» проверяйте, есть ли идентификатор запланированного кадра анимации. Если есть — игра идёт, нужно отменить кадр, чтобы `renderLoop` больше не вызывался (пауза). Если идентификатора нет — мы на паузе, вызываем `requestAnimationFrame`, чтобы возобновить игру.

Так как цикл ведёт JavaScript, менять исходники Rust не нужно.

Введём переменную `animationId` для идентификатора из `requestAnimationFrame`. Когда кадр не запланирован, в ней хранится `null`.

```js
let animationId = null;

// Как раньше, но результат `requestAnimationFrame` сохраняем в `animationId`.
const renderLoop = () => {
    drawGrid();
    drawCells();

    universe.tick();

    animationId = requestAnimationFrame(renderLoop);
};
```

В любой момент по значению `animationId` видно, на паузе игра или нет:

```js
const isPaused = () => {
    return animationId === null;
};
```

По клику на кнопку проверяем паузу или воспроизведение и либо снова запускаем `renderLoop`, либо отменяем следующий кадр. Заодно обновляем подпись на кнопке так, чтобы она показывала действие при следующем клике.

```js
const playPauseButton = document.getElementById(
    'play-pause'
);

const play = () => {
    playPauseButton.textContent = '⏸';
    renderLoop();
};

const pause = () => {
    playPauseButton.textContent = '▶';
    cancelAnimationFrame(animationId);
    animationId = null;
};

playPauseButton.addEventListener('click', (event) => {
    if (isPaused()) {
        play();
    } else {
        pause();
    }
});
```

Раньше мы запускали анимацию прямым вызовом `requestAnimationFrame(renderLoop)`; замените это на вызов `play`, чтобы с самого начала на кнопке была правильная подпись.

```diff
// Раньше здесь было `requestAnimationFrame(renderLoop)`.
play();
```

Обновите [http://localhost:8080/](http://localhost:8080/) — по кнопке можно ставить игру на паузу и снимать с паузы.

## Переключение состояния клетки по событию `"click"`

Раз пауза есть, добавим изменение клеток по клику.

Переключить клетку — значит сменить состояние «жива» ↔ «мёртва». Добавьте метод `toggle` для `Cell` в `wasm-game-of-life/src/lib.rs`:

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

Чтобы переключить клетку в заданной строке и столбце, переводим пару (строка, столбец) в индекс в векторе клеток и вызываем `toggle` у этой клетки:

```rust
/// Публичные методы, экспортируемые в JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

Метод объявлен в блоке `impl` с атрибутом `#[wasm_bindgen]`, чтобы его можно было вызывать из JavaScript.

В `wasm-game-of-life/www/index.js` подписываемся на клики по `<canvas>`: переводим координаты клика из системы страницы в координаты холста, затем в строку и столбец, вызываем `toggle_cell` и перерисовываем сцену.

```js
canvas.addEventListener('click', (event) => {
    const boundingRect = canvas.getBoundingClientRect();

    const scaleX = canvas.width / boundingRect.width;
    const scaleY = canvas.height / boundingRect.height;

    const canvasLeft =
        (event.clientX - boundingRect.left) * scaleX;
    const canvasTop =
        (event.clientY - boundingRect.top) * scaleY;

    const row = Math.min(
        Math.floor(canvasTop / (CELL_SIZE + 1)),
        height - 1
    );
    const col = Math.min(
        Math.floor(canvasLeft / (CELL_SIZE + 1)),
        width - 1
    );

    universe.toggle_cell(row, col);

    drawGrid();
    drawCells();
});
```

Соберите проект командой `wasm-pack build` в каталоге `wasm-game-of-life`, снова откройте [http://localhost:8080/](http://localhost:8080/) — можно рисовать свои паттерны кликами по клеткам.

## Упражнения

-   Добавьте виджет [`<input type="range">`](https://developer.mozilla.org/ru/docs/Web/HTML/Element/input/range), чтобы задавать, сколько тиков выполняется за один кадр анимации.
-   Кнопка сброса вселенной в случайное начальное состояние по клику. Ещё одна кнопка — сброс во «все клетки мёртвы».
-   По `Ctrl + клик` вставляйте [планер (глайдер)](<https://ru.wikipedia.org/wiki/%D0%9F%D0%BB%D0%B0%D0%BD%D0%B5%D1%80_(%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D1%8F_%D0%BA%D0%BB%D0%B5%D1%82%D0%BE%D1%87%D0%BD%D0%BE%D0%B3%D0%BE_%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82%D0%B0)>), центрированный на целевой клетке. По `Shift + клик` — [пульсар](<https://en.wikipedia.org/wiki/Pulsar_(Conway%27s_Life)>) (классический осциллятор в игре жизни).
