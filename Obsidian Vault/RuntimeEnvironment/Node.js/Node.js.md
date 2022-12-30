- [[JavaScript]]

# About Node.js®

As an asynchronous event-driven JavaScript runtime, Node.js is designed to build scalable network applications. In the following "hello world" example, many connections can be handled concurrently. Upon each connection, the callback is fired, but if there is no work to be done, Node.js will sleep.

# Как на самом деле работает Node.js

Node.js использует два вида потоков:

-   основной поток, обрабатываемый циклом событий (_Event Loop_),
-   несколько [вспомогательных потоков](Multithreading.md) в пуле воркеров.

Цикл обработки событий — это механизм, который принимает callback-функции и регистрирует их для выполнения в определённый момент в будущем. Он работает в том же потоке, что и сам код JavaScript. Когда операция блокирует поток, цикл событий также блокируется.

Пул воркеров — модель исполнения, вызывающая и обрабатывающая отдельные потоки. Затем они синхронно выполняют задачу и возвращают результат в цикл обработки событий. После цикл вызывает callback-функцию с указанным результатом.

Если коротко, то пул воркеров может заниматься асинхронными операциями ввода-вывода — прежде всего, взаимодействем с системным диском и сетью. Эта модель исполнения в основном используется модулями вроде `fs` (требовательного к скорости ввода-вывода) или `crypto` (требовательного к CPU). Пул воркеров реализован в [libuv](http://docs.libuv.org/en/v1.x/), что приводит к небольшой задержке всякий раз, когда Node требует связи между JavaScript и C ++, но эта задержка едва ощутима.

Используя оба эти механизма, можно написать следующий код:

```javascript
fs.readFile(path.join(__dirname, './package.json'), (err, content) => {
 if (err) {
   return null;
 }

 console.log(content.toString());
});
```

Модуль `fs` указывает пулу воркеров использовать один из его потоков для чтения содержимого файла и уведомления цикла обработки событий, когда это будет сделано. Цикл принимает предоставленную callback-функцию и выполняет её с содержимым файла.

Выше приведён пример неблокирующего кода. Пул воркеров прочитает файл и вызовет предоставленную функцию с результатом. Поскольку пул имеет собственные потоки, цикл обработки событий может продолжать исполнение в обычном режиме во время чтения файла.

Всё работает, пока нет необходимости синхронно выполнять какую-то сложную операцию. Любая функция, выполнение которой занимает слишком много времени, блокирует поток. Если в приложении много таких функций, оно может значительно снизить производительность сервера или вообще заморозить его работу в целом. В этом случае нет способа делегировать работу пулу воркеров.

Области, требующие сложных вычислений, — искусственный интеллект, машинное обучение или большие данные— не могли эффективно использовать Node.js из-за операций, блокирующих основной (и единственный) поток, что делало сервер неотзывчивым. Так было до появления Node.js v10.5.0, в котором была добавлена поддержка нескольких потоков.

Более подробно о [[BlockingProblem]]

## [[Event Loop]]
## [[Multithreading]]