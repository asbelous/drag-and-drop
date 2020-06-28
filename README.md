# Туториал. Список задач с drag &amp; drop by HTMLAcademy

В этом туториале мы рассмотрим, как реализовать эффект drag & drop на ванильном JavaScript. Дословный перевод с английского — «потяни и брось» — отражает суть эффекта, это хорошо знакомое любому пользователю перетаскивание элементов интерфейса. Drag & drop может понадобиться в разных ситуациях — например, в таких:

    Простое визуальное изменение положения элемента.
    Сортировка элементов с помощью перетаскивания. Пример — сортировка карточек задач в таск-трекере.
    Изменение контекста элемента. Пример — перенос задачи в таск трекере из одного списка в другой.
    Перемещение локальных файлов в окно браузера.

Мы разберём drag & drop на примере сортировки. Для этого создадим интерактивный список задач.

## HTML Drag and Drop API

В стандарте HTML5 есть API, который позволяет реализовать эффект drag & drop. Он даёт возможность с помощью специальных событий контролировать захват элемента на странице мышью и его перемещение в новое положение. Рассмотрим этот API подробнее.

По умолчанию перемещаться могут только ссылки, изображения и выделенные фрагменты. Если начать перетаскивать их, появится фантомная копия, которая будет следовать за курсором. Чтобы добавить возможность перетаскивания другим элементам, нужно задать атрибуту draggable значение true.

<div draggable="true">Draggable element</div>

Далее для реализации перемещения используется ряд событий, которые срабатывают на разных этапах. Полный список есть на MDN, а мы рассмотрим основные.

    drag — срабатывает каждые несколько сотен миллисекунд, пока элемент перетаскивается.
    dragstart — срабатывает в момент начала перетаскивания элемента.
    dragend — срабатывает в момент, когда перетаскивание элемента завершено.
    dragover — срабатывает каждые несколько сотен миллисекунд, пока перетаскиваемый элемент находится над зоной, в которую может быть сброшен.
    drop — срабатывает в тот момент, когда элемент будет брошен, если он может быть перемещён в текущую зону.

При успешном перемещении элемент должен оказаться на новом месте. Но по умолчанию большинство областей на странице недоступны для сброса. Чтобы создать область, в которую элементы могут быть сброшены, необходимо слушать событие dragover или drop на нужном элементе и отменять действие по умолчанию с помощью метода preventDefault. Тогда стандартное поведение будет переопределено — перетаскивание и сброс в эту область станут возможными. Рассмотрим на примере чуть позже.

Это далеко не все возможности API. Также в нём есть несколько интерфейсов, которые помогают получать доступ к данным перетаскиваемого элемента и изменять их. Например, существует объект DataTransfer, который кроме всего прочего хранит информацию о локальных файлах, если они перетаскиваются. В этом туториале нам не понадобится использовать эти возможности, но без DataTransfer не обойтись, если нужно, например, загружать файлы с компьютера и считывать данные, чтобы затем производить с ними какие-либо манипуляции. Подробно об этом на MDN.

Приступим к созданию нашего списка задач и рассмотрим на примере, как работать с HTML Drag and Drop API.

## Вёрстка и стилизация списка задач

Список будет состоять из нескольких задач и заголовка. Для начала создадим разметку. Здесь всё просто — если речь идёт о списке, значит нужен тег ul.

<section class="tasks">
  <h1 class="tasks__title">To do list</h1>

  <ul class="tasks__list">
    <li class="tasks__item">learn HTML</li>
    <li class="tasks__item">learn CSS</li>
    <li class="tasks__item">learn JavaScript</li>
    <li class="tasks__item">learn PHP</li>
    <li class="tasks__item">stay alive</li>
  </ul>
</section>

Теперь добавим элементам базовую стилизацию:

body {
  font-family: "Tahoma", sans-serif;
  font-size: 18px;
  line-height: 25px;
  color: #164a44;
  background-color: #b2d9d0;
}

.tasks__title {
    margin: 50px 0 20px 0;
    text-align: center;
    text-transform: uppercase;
 }

.tasks__list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.tasks__item {
  transition: background-color 0.5s;
  margin-bottom: 10px;
  padding: 5px;
  text-align: center;
  border: 2px dashed #b2d9d0;
  border-radius: 10px;
  cursor: move;
  background-color: #dff2ef;
  transition: background-color 0.5s;
}

.tasks__item:last-child {
  margin-bottom: 0;
}

.selected {
  opacity: 0.6;
}

Здесь стоит обратить внимание на тип курсора, который мы указали — move. Все элементы списка смогут перемещаться, а с помощью курсора move мы подсказываем пользователю, что есть такая возможность.

Также мы задали стилизацию для класса selected, который чуть позже будем добавлять программно при взаимодействии с элементом.

## Реализация drag & drop

### Шаг 1. Разрешим перетаскивание элементов

Переходим к JavaScript. В первую очередь присвоим элементам упомянутый ранее атрибут draggable со значением true, чтобы разрешить задачам перемещаться. Это можно сделать прямо в разметке или с помощью JavaScript.

const tasksListElement = document.querySelector(`.tasks__list`);

const taskElements = tasksListElement.querySelectorAll(`.tasks__item`);

// Перебираем все элементы списка и присваиваем нужное значение

for (const task of taskElements) {
  task.draggable = true;
}

Уже сейчас перетаскивание доступно для элементов, но пока это выражается только в появлении фантомной копии. Своего положения элементы не меняют, добавим перемещение чуть позже.

### Шаг 2. Добавим реакцию на начало и конец перетаскивания

Будем отслеживать события dragstart и dragend на всём списке. В начале перетаскивания будем добавлять класс selected элементу списка, на котором было вызвано событие. После окончания перетаскивания будем удалять этот класс.

tasksListElement.addEventListener(`dragstart`, (evt) => {
  evt.target.classList.add(`selected`);
})

tasksListElement.addEventListener(`dragend`, (evt) => {
  evt.target.classList.remove(`selected`);
});
