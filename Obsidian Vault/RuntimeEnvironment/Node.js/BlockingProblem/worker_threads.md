[Source](https://tproger.ru/translations/guide-to-threads-in-node-js/)

## Знакомство с worker_threads

Модуль `worker_threads` — это пакет, который позволяет создавать полнофункциональные многопоточные приложения Node.js.

Потоковый воркер (_thread worker_) — фрагмент кода (обычно извлекаемый из файла), созданный в отдельном потоке.

Для использования потоковых воркеров нужно импортировать модуль `worker_threads`. Начнём с создания функции, которая поможет создавать эти воркеры, а также рассмотрим их свойства.

```javascript
type WorkerCallback = (err: any, result?: any) => any;

export function runWorker(path: string, cb: WorkerCallback, workerData: object | null = null) {
 const worker = new Worker(path, { workerData });

 worker.on('message', cb.bind(null, null));
 worker.on('error', cb);

 worker.on('exit', (exitCode) => {
   if (exitCode === 0) {
     return null;
   }

   return cb(new Error(`Worker has stopped with code ${exitCode}`));
 });

 return worker;
}
```

Для создания потокового воркера необходимо создать экземпляр класса `Worker`. В первом аргументе указываем путь к файлу, который содержит код воркера; во втором предоставляем объект, содержащий свойство с именем `workerData`. Это те данные, к которым поток будет иметь доступ при запуске, если того хочет разработчик.

Обратите внимание: независимо от того, используете ли вы сам JavaScript или что-то, что в него транспилируется (например TypeScript), путь всегда должен ссылаться на файлы с расширениями `.js` или `.mjs`.

Также стоит указать, почему используется callback-функция вместо возвращения промиса (promise), который будет передавать результат в `resolve` при запуске события `message`. Это связано с возможностью потоковых воркеров отправлять много событий `message`, а не только одно.

Связь между потоками основана на событиях. Это означает, что надо настроить обработчики, которые будут вызываться после отправки потоком данного события.

Рассмотрим наиболее распространённые события.

```javascript
worker.on('error', (error) => {});
```

Событие `error` генерируется, когда внутри воркера возникает необработанное исключение. Затем поток завершается, а ошибка становится первым аргументом в callback.

```javascript
worker.on('exit', (exitCode) => {});
```

`Exit` генерируется, когда воркер заканчивает своё выполнение. Если `process.exit()` вызывается внутри потока, `exitCode` будет предоставлен в callback. Если поток прерывается с помощью `worker.terminate()`, код будет `1`.

```javascript
worker.on('online', () => {});
```

`Online` генерируется, когда воркер прекращает парсинг кода JavaScript и начинает его выполнение. Это событие используется нечасто, но в определённых случаях оно может быть информативным.

```javascript
worker.on('message', (data) => {});
```

`Message` генерируется, когда воркер отправляет данные в родительский поток.

## Обмен данными между потоками

Для отправки данных другому потоку используется метод `port.postMessage()`. Он имеет следующую сигнатуру:

```javascript
port.postMessage(data[, transferList])
```

Объект `port` может быть или экземпляром `parentPort`, или экземпляром `MessagePort` — подробнее об этом позже.

### Аргумент data

Первый аргумент данных — назовём его `data` — это объект, который копируется в другой поток. Он может содержать всё, что поддерживает алгоритм копирования.

Данные копируются [алгоритмом структурированного клонирования](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm).

Алгоритм не копирует функции, ошибки, дескрипторы свойств или цепочки прототипов. Следует также отметить, что копирование объектов таким способом отличается от JSON, потому что он может содержать циклические ссылки и типизированные массивы, а JSON не может.

Поддерживая копирование типизированных массивов, алгоритм позволяет разделять память между потоками.

### Разделение памяти между потоками

Считается, что модули вроде `cluster` или `child_process` используют потоки уже давно. Это одновременно и верно и нет.

`Cluster` может создавать несколько процессов Node.js с одним главным процессом, маршрутизирующим запросы между ними. Кластеризация приложения позволяет эффективно увеличить пропускную способность сервера. Однако нельзя создать отдельный поток с модулем `cluster`.

Модуль `child_process` может создавать любой исполняемый файл независимо от типа файла. В этом модуле отсутствуют некоторые важные функции, которые есть у `worker_threads`.

Потоковые воркеры являются более лёгкими и имеют тот же идентификатор процесса, что и их родительские потоки. Ещё они могут использовать память совместно со своими родительскими потоками. Это позволяет воркерам избежать сериализации больших входных данных и, как следствие, отправлять данные вперёд и назад более эффективно.

Рассмотрим пример разделения памяти между потоками. Чтобы память была разделена, экземпляры `ArrayBuffer` или `SharedArrayBuffer` должны быть отправлены другому потоку в качестве аргумента `data` или внутри него.

Пример воркера, который разделяет память со своим родительским потоком:

```javascript
import { parentPort } from 'worker_threads';

parentPort.on('message', () => {
 const numberOfElements = 100;
 const sharedBuffer = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * numberOfElements);
 const arr = new Int32Array(sharedBuffer);

 for (let i = 0; i < numberOfElements; i += 1) {
   arr[i] = Math.round(Math.random() * 30);
 }

 parentPort.postMessage({ arr });
});
```

Создаётся экземпляр `SharedArrayBuffer` с размером памяти, необходимым для хранения ста 32-битных целых чисел. Затем создаётся экземпляр `Int32Array`, который будет использовать буфер для хранения его структуры. После массив заполняется некоторыми случайными числами и отправляется в родительский поток.

В родительском потоке:

```javascript
import path from 'path';

import { runWorker } from '../run-worker';

const worker = runWorker(path.join(__dirname, 'worker.js'), (err, { arr }) => {
 if (err) {
   return null;
 }

 arr[0] = 5;
});

worker.postMessage({});
```

Меняя значение `arr[0]` на 5, фактически изменяем его в обоих потоках.

При разделении памяти есть риск изменить значение в одном потоке, изменив его в другом. Но вместе с этим появляется хорошая особенность: значение не нужно сериализовывать, чтобы оно было доступно в другом потоке. Это значительно повышает эффективность. Просто не забывайте правильно управлять ссылками на данные, чтобы те в свою очередь не оставляли за собой мусор после завершения работы с ними.

Зачастую гораздо удобнее передавать между потоками не массив, а объект. Но, к сожалению, не существует `SharedObjectBuffer` или чего-либо подобного, но можно самим [создать похожую структуру](https://stackoverflow.com/questions/51053222/nodejs-worker-threads-shared-object-store).

### Аргумент TransferList

`TransferList` может содержать только `ArrayBuffer` и `MessagePort`. После передачи в другой поток их больше нельзя использовать в отправляющем потоке. Память перемещается в другой поток и, следовательно, недоступна в отправляющем.

Пока нет канала связи, нельзя передавать сетевые сокеты, включая их в `TransferList`.

### Создание канала связи

Связь между потоками осуществляется через порты, которые являются экземплярами класса `MessagePort`. Они обеспечивают эту связь на основе событий.

Есть два способа использования портов для связи между потоками. Первый используется по умолчанию и проще, чем второй. В код воркера импортируется объект с именем `parentPort` из модуля `worker_threads` и используется `.postMessage()` для отправки сообщений в родительский поток.

Пример:

```javascript
import { parentPort } from 'worker_threads';
const data = {
 // ...
};

parentPort.postMessage(data);
```

`parentPort` — это экземпляр `MessagePort`, который Node.js создал “за кулисами”, чтобы обеспечить связь с родительским потоком. Таким образом, можно общаться между потоками, используя объекты `parentPort` и `worker`.

Второй способ связи между потоками — создать `MessageChannel` и отправить его воркеру. Вот как можно создать новый `MessagePort` и поделиться им с потоковым воркером:

```javascript
import path from 'path';
import { Worker, MessageChannel } from 'worker_threads';

const worker = new Worker(path.join(__dirname, 'worker.js'));

const { port1, port2 } = new MessageChannel();

port1.on('message', (message) => {
 console.log('message from worker:', message);
});

worker.postMessage({ port: port2 }, [port2]);
```

После создания `port1` и `port2` настраиваем обработчики событий на `port1` и отправляем `port2` воркеру. Необходимо включить его в файл `TransferList`, чтобы он был передан рабочей стороне.

Теперь внутри воркера:

```javascript
import { parentPort, MessagePort } from 'worker_threads';

parentPort.on('message', (data) => {
 const { port }: { port: MessagePort } = data;

 port.postMessage('heres your message!');
});
```

Таким образом, используется порт, который был отправлен родительским потоком.

Использование `parentPort` тоже является правильным подходом, но лучше создать новый `MessagePort` с экземпляром `MessageChannel,` а затем поделиться им с созданным воркером.

Обратите внимание, в примерах ниже для простоты используется `parentPort`.

### Два способа использования воркеров

Первый — создать воркер, выполнить его код и отправить результат в родительский поток. При таком подходе каждый раз, когда появляется новая задача, надо заново создавать воркер.

Второй способ — создать воркер и настроить обработчики для события `message`. Каждый раз при запуске это событие выполняет свою работу и отправляет результат обратно в родительский поток, который сохраняет воркер для последующего использования.

Документация Node.js рекомендует второй подход, поскольку много усилий необходимо для создания потокового воркера, который требует создания виртуальной машины, парсинга и выполнения кода. Этот метод также намного эффективнее, чем постоянно создающиеся воркеры.

Такой подход называется пулом воркеров. Создаётся пул и воркеры находятся в ожидании события `message`, которое нужно для выполнения задания.

Пример файла, содержащего воркер, который создаётся, выполняется, а затем закрывается:

```javascript
import { parentPort } from 'worker_threads';

const collection = [];

for (let i = 0; i < 10; i += 1) {
 collection[i] = i;
}

parentPort.postMessage(collection);
```

После отправки `collection` в родительский поток, воркер просто завершается.

А вот пример воркера, который может ждать в течение длительного периода времени, прежде чем ему будет дано задание:

```javascript
import { parentPort } from 'worker_threads';

parentPort.on('message', (data: any) => {
 const result = doSomething(data);

 parentPort.postMessage(result);
});
```

## Полезные свойства модуля worker_threads

**isMainThread**

Свойство имеет значение `true`, когда не работает внутри потока воркера. Если есть необходимость, можно включить простой оператор `if` в начале файла, чтобы убедиться, что он запускается только как воркер.

**workerData**

Несёт в себе данные, включённые в конструктор воркера созданным потоком.

```javascript
const worker = new Worker(path, { workerData });
```

В потоке воркера:

```javascript
import { workerData } from 'worker_threads';
console.log(workerData.property);
```

**parentPort**

Экземпляр `MessagePort`, используется для связи с родительским потоком.

**threadId**

Уникальный идентификатор, присвоенный воркеру.

## Реализация setTimeout

`setTimeout` — это бесконечный цикл, который прерывает выполнение приложения. На практике он проверяет на каждой итерации, меньше ли сумма начальной даты и заданного количества миллисекунд, чем фактическая дата.

```javascript
import { parentPort, workerData } from 'worker_threads';

const time = Date.now();

while (true) {
 if (time + workerData.time <= Date.now()) {
   parentPort.postMessage({});
   break;
 }
}
```

Эта конкретная реализация создаёт поток, выполняет его код и затем завершает работу.

Реализуем код, который будет использовать этот воркер. Создадим стейт, в котором будут отслеживаться созданные воркеры:

```javascript
const timeoutState: { [key: string]: Worker } = {};
```

Функция, которая отвечает за создание потоковых воркеров и хранит их в стейт:

```javascript
export function setTimeout(callback: (err: any) => any, time: number) {
 const id = uuidv4();

 const worker = runWorker(
   path.join(__dirname, './timeout-worker.js'),
   (err) => {
     if (!timeoutState[id]) {
       return null;
     }

     timeoutState[id] = null;

     if (err) {
       return callback(err);
     }

     callback(null);
   },
   {
     time,
   },
 );

 timeoutState[id] = worker;

 return id;
}
```

Используем пакет UUID для создания уникального идентификатора воркера, затем задействуем определённую ранее вспомогательную функцию `runWorker()`, чтобы получить воркер. Передаём ему callback-функцию, которая запускается после отправки воркером некоторых данных. Сохраняем воркер в стейт и возвращаем `id`.

Внутри callback-функции нужно проверить, существует ли воркер в стейте, потому что есть возможность отменить его с помощью `cancelTimeout()`. Если он существует, удаляем его из стейта и вызываем callback, переданный в функцию `setTimeout()`.

Функция `cancelTimeout()` использует метод `.terminate()`, чтобы принудительно остановить воркер и удалить его из стейта:

```javascript
export function cancelTimeout(id: string) {
 if (timeoutState[id]) {
   timeoutState[id].terminate();

   timeoutState[id] = undefined;

   return true;
 }

 return false;
}
```

Прим. если вам интересно, есть [реализация](https://github.com/maciejcieslar/threads-nodejs/blob/master/src/timeout/timeout.ts#L64) метода `setInterval()`. Но он не имеет ничего общего с потоками (повторно используется код `setTimeout()`). Кроме того, существует небольшой тестовый код для проверки, насколько такой подход отличается от исходного. Вы можете просмотреть код [здесь](https://github.com/maciejcieslar/threads-nodejs/blob/master/src/timeout/index.ts#L13). Результаты:

```javascript
native setTimeout { ms: 7004, averageCPUCost: 0.1416 }
worker setTimeout { ms: 7046, averageCPUCost: 0.308 }
```

Видно, что в `setTimeout()` есть небольшая задержка — около 40 мс — из-за создаваемого воркера. Средняя стоимость процессора также немного выше, но ничего страшного в этом нет (стоимость процессора — это среднее значение загрузки процессора за всё время процесса).

Если бы можно было повторно использовать воркеры, задержка и загрузка ЦП снизилась бы. Поэтому рассмотрим, как реализовать собственный пул воркеров.

## Реализация пула воркеров

Пул воркеров — это заданное количество ранее созданных воркеров, которые ожидают событие `message`. Как только событие происходит, воркеры выполняют работу и отправляют результат обратно.

Вот как можно создать пул воркеров из восьми рабочих потоков:

```javascript
const pool = new WorkerPool(path.join(__dirname, './test-worker.js'), 8);
```

Если вы знакомы с [ограничением параллельных операций](https://medium.freecodecamp.org/how-to-limit-concurrent-operations-in-javascript-b57d7b80d573), то знаете, что логика здесь почти одинакова.

Из фрагмента выше видно, конструктору `WorkerPool` передаётся количество воркеров и путь для их появления.

```javascript
export class WorkerPool<T, N> {
 private queue: QueueItem<T, N>[] = [];
 private workersById: { [key: number]: Worker } = {};
 private activeWorkersById: { [key: number]: boolean } = {};

 public constructor(public workerPath: string, public numberOfThreads: number) {
   this.init();
 }
}
```

Здесь есть дополнительные свойства вроде `workerById` и `activeWorkersById`, в которых можно сохранить существующие воркеры и их идентификаторы соответственно. Также есть `queue` (очередь), в которой можно сохранять объекты со следующей структурой:

```javascript
type QueueCallback = (err: any, result?: N) => void;

interface QueueItem<T, N> {
 callback: QueueCallback;
 getData: () => T;
}
```

`callback` — callback-функция в Node по умолчанию с ошибкой в качестве первого аргумента и возможным результатом в качестве второго. `getData` — это функция, передаваемая методу `.run()` пула воркеров (поясняется ниже), которая вызывается после начала обработки элемента. Данные, возвращаемые функцией `getData()`, будут переданы в рабочий поток.

Внутри метода `.init()` создаём воркеры и сохраняем их в стейтах:

```javascript
private init() {
  if (this.numberOfThreads < 1) {
    return null;
  }

  for (let i = 0; i < this.numberOfThreads; i += 1) {
    const worker = new Worker(this.workerPath);

    this.workersById[i] = worker;
    this.activeWorkersById[i] = false;
  }
}
```

Для избежания бесконечных циклов нужно убедиться, что количество потоков больше 1. Создаём необходимое число воркеров и сохраняем их по индексу в стейте `workerById`. Также сохраняем информацию, работают ли они в настоящее время, в стейте `activeWorkersById`, который всегда по умолчанию имеет значение `false`.

Реализуем метод `.run()` для настройки задачи, которая будет запущена, как только воркер станет доступен.

```javascript
public run(getData: () => T) {
  return new Promise((resolve, reject) => {
    const availableWorkerId = this.getInactiveWorkerId();

    const queueItem: QueueItem<T, N> = {
      getData,
      callback: (error, result) => {
        if (error) {
          return reject(error);
        }
return resolve(result);
      },
    };

    if (availableWorkerId === -1) {
      this.queue.push(queueItem);

      return null;
    }

    this.runWorker(availableWorkerId, queueItem);
  });
}
```

Внутри функции, переданной в промис, проверяем, есть ли доступный для обработки данных воркер, вызывая `.getInactiveWorkerId()`:

```javascript
private getInactiveWorkerId(): number {
  for (let i = 0; i < this.numberOfThreads; i += 1) {
    if (!this.activeWorkersById[i]) {
      return i;
    }
  }

  return -1;
}
```

Создаём `queueItem`, в котором сохраняем переданную методу `.run()` функцию `getData()` в качестве callback. В этом callback разрешаем (`resolve`) или отклоняем (`reject`) промис в зависимости от того, передал ли воркер callback.

Если значение `availableWorkerId` равно -1, доступного воркера нет. В этом случае добавляем `queueItem` в `queue`. Если есть доступный воркер, вызываем метод `.runWorker()` для его выполнения.  
В методе `.runWorker()` в стейте `activeWorkersById` необходимо установить, что воркер в данный момент используется. Также нужно настроить обработчики для событий `message` и `error` (после очистить их). И, наконец, отправить данные воркеру.

```javascript
const messageCallback = (result: N) => {
   queueItem.callback(null, result);

   cleanUp();
 };

 const errorCallback = (error: any) => {
   queueItem.callback(error);

   cleanUp();
 };

 const cleanUp = () => {
   worker.removeAllListeners('message');
   worker.removeAllListeners('error');

   this.activeWorkersById[workerId] = false;

   if (!this.queue.length) {
     return null;
   }

   this.runWorker(workerId, this.queue.shift());
 };

 worker.once('message', messageCallback);
 worker.once('error', errorCallback);

 worker.postMessage(await queueItem.getData());
}
```

Используя переданный `workerId`, получаем ссылку на воркер из стейта `workerById`. Внутри `activeWorkersById` устанавливаем в свойстве `[workerId]` значение `true`. Таким образом будет известно, что больше ничего не нужно запускать, пока воркер занят.

Создаём `messageCallback()` и `errorCallback()` для вызова событий `message` и `error` соответственно. Регистрируем указанные функции для обработки события и отправки данных воркеру.

Внутри функций вызываем callback в `queueItem`, а затем вызываем функцию `cleanUp()`. Убеждаемся, что обработчики событий удаляются, т. к. один и тот же воркер используется многократно. Если не удалить обработчики, произойдёт утечка памяти (память медленно исчерпается).

В стейте `activeWorkersById` устанавливаем для свойства `[workerId]` значение `false` и проверяем, пуста ли очередь. Если это не так, удаляем первый элемент из `queue` и снова вызываем воркер с другим `queueItem`.

Создадим воркер, который выполняет некоторые вычисления после получения данных в событии `message`:

```javascript
import { isMainThread, parentPort } from 'worker_threads';

if (isMainThread) {
 throw new Error('Its not a worker');
}

const doCalcs = (data: any) => {
 const collection = [];

 for (let i = 0; i < 1000000; i += 1) {
   collection[i] = Math.round(Math.random() * 100000);
 }

 return collection.sort((a, b) => {
   if (a > b) {
     return 1;
   }

   return -1;
 });
};

parentPort.on('message', (data: any) => {
 const result = doCalcs(data);

 parentPort.postMessage(result);
});
```

Потоковый воркер создаёт массив из 1 миллиона случайных чисел, а затем сортирует их.

Пример простого использования пула воркеров:

```javascript
const pool = new WorkerPool<{ i: number }, number>(path.join(__dirname, './test-worker.js'), 8);

const items = [...new Array(100)].fill(null);

Promise.all(
 items.map(async (_, i) => {
   await pool.run(() => ({ i }));

   console.log('finished', i);
 }),
).then(() => {
 console.log('finished all');
});
```

Всё начиналось с создания пула из восьми воркеров. Затем был создан массив из 100 элементов и для каждого элемента запускалась задача в пуле воркеров. Первые восемь задач были выполнены немедленно, а остальные помещены в очередь и выполнены постепенно. Благодаря использованию пула воркеров не нужно каждый раз создавать воркер, что значительно повышает эффективность.

## Заключение

`worker_threads` предоставляет простой способ добавить поддержку многопоточности в приложения. Передавая тяжёлые CPU-вычисления другим потокам, можно значительно увеличить пропускную способность сервера. Благодаря официальной поддержке потоков можно ожидать, что всё больше разработчиков и инженеров из различных областей (ИИ, машинное обучение и большие данные) начнут использовать Node.js.