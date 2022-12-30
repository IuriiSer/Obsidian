# Proccesses and Threads

[Source](https://www.digitalocean.com/community/tutorials/how-to-use-multithreading-in-node-js)

### Introduction

[[Node.js]] runs JavaScript code in a single thread, which means that your code can only do one task at a time. However, Node.js itself is multithreaded and provides hidden threads through the [`libuv`](https://libuv.org/) library, which handles I/O operations like reading files from a disk or network requests. Through the use of hidden threads, Node.js provides asynchronous methods that allow your code to make I/O requests without blocking the main thread.

Although Node.js has hidden threads, you cannot use them to offload CPU-intensive tasks, such as complex calculations, image resizing, or video compression. Since JavaScript is single-threaded when a CPU-intensive task runs, it blocks the main thread and no other code executes until the task completes. Without using other threads, the only way to speed up a CPU-bound task is to increase the processor speed.

However, in recent years, CPUs haven’t been getting faster. Instead, computers are shipping with extra cores, and it’s now more common for computers to have 8 or more cores. Despite this trend, your code will not take advantage of the extra cores on your computer to speed up CPU-bound tasks or avoid breaking the main thread because JavaScript is single-threaded.

To remedy this, Node.js introduced the [`worker-threads`](https://nodejs.org/api/worker_threads.html) module, which allows you to create threads and execute multiple JavaScript tasks in parallel. Once a thread finishes a task, it sends a message to the main thread that contains the result of the operation so that it can be used with other parts of the code. The advantage of using worker threads is that CPU-bound tasks don’t block the main thread and you can divide and distribute a task to multiple workers to optimize it.

In this tutorial, you’ll create a Node.js app with a CPU-intensive task that blocks the main thread. Next, you will use the `worker-threads` module to offload the CPU-intensive task to another thread to avoid blocking the main thread. Finally, you will divide the CPU-bound task and have four threads work on it in parallel to speed up the task.

## Understanding Processes and Threads

Before you start writing CPU-bound tasks and offloading them to separate threads, you first need to understand what processes and threads are, and the differences between them. Most importantly, you’ll review how the processes and threads execute on a single or multi-core computer system.

### Process

A process is a running program in the operating system. It has its own memory and cannot see nor access the memory of other running programs. It also has an instruction pointer, which indicates the instruction currently being executed in a program. Only one task can be executed at a time.

To understand this, you will create a Node.js program with an infinite loop so that it doesn’t exit when run.

Using `nano`, or your preferred text editor, create and open the `process.js` file:

```
nano process.js
```

In your `process.js` file, enter the following code:

multi-threading_demo/process.js

```
const process_name = process.argv.slice(2)[0];

count = 0;
while (true) {
  count++;
  if (count == 2000 || count == 4000) {
    console.log(`${process_name}: ${count}`);
  }
}

```

In the first line, the `process.argv` property returns an array containing the program command-line arguments. You then attach JavaScript’s `slice()` method with an argument of `2` to make a shallow copy of the array from index 2 onwards. Doing so skips the first two arguments, which are the Node.js path and the program filename. Next, you use the bracket notation syntax to retrieve the first argument from the sliced array and store it in the `process_name` variable.

After that, you define a `while` loop and pass it a `true` condition to make the loop run forever. Within the loop, the `count` variable is incremented by `1` during each iteration. Following this is an `if` statement that checks whether `count` is equal to `2000` or `4000`. If the condition evaluates to true, `console.log()` method logs a message in the terminal.

Save and close your file using `CTRL+X`, then press `Y` to save the changes.

Run the program using the `node` command:

```
node process.js A &
```

`A` is a command-line argument that is passed to the program and stored in the `process_name` variable. The `&` at end the allows the Node program to run in the background, which lets you enter more commands in the shell.

When you run the program, you will see output similar to the following:

```
Output[1] 7754

A: 2000
A: 4000
```

The number `7754` is a process ID that the operating system assigned to it. `A: 2000` and `A: 4000` are the program’s output.

When you run a program using the `node` command, you create a process. The operating system allocates memory for the program, locates the program executable on your computer’s disk, and loads the program into memory. It then assigns it a process ID and begins executing the program. At that point, your program has now become a process.

When the process is running, its process ID is added to the process list of the operating system and can be seen with tools like [`htop`](https://htop.dev/), [`top`](https://en.wikipedia.org/wiki/Top_(software)), or [`ps`](https://man7.org/linux/man-pages/man1/ps.1.html). The tools provide more details about the processes, as well as options to stop or prioritize them.

To get a quick summary of a Node process, press `ENTER` in your terminal to get the prompt back. Next, run the `ps` command to see the Node processes:

```
ps |grep node
```

The `ps` command lists all processes associated with the current user on the system. The pipe operator `|` to pass all the `ps` output to the [`grep`](https://man7.org/linux/man-pages/man1/egrep.1.html) filters the processes to list only Node processes.

Running the command will yield output similar to the following:

```
Output7754 pts/0    00:21:49 node
```

You can create countless processes out of a single program. For example, use the following command to create three more processes with different arguments and put them in the background:

```
node process.js B & node process.js C & node process.js D &
```

In the command, you created three more instances of the `process.js` program. The `&` symbol puts each process in the background.

Upon running the command, the output will look similar to the following (although the order might differ):

```
Output[2] 7821
[3] 7822
[4] 7823

D: 2000
D: 4000
B: 2000
B: 4000
C: 2000
C: 4000
```

As you can see in the output, each process logged the process name into the terminal when the count reached `2000` and `4000`. Each process is not aware of any other process running: process `D` isn’t aware of process `C`, and vice versa. Anything that happens in either process will not affect other Node.js processes.

If you examine the output closely, you will see that the order of the output isn’t the same order you had when you created the three processes. When running the command, the processes arguments were in order of `B`, `C`, and `D`. But now, the order is `D`, `B`, and `C`. The reason is that the OS has scheduling algorithms that decide which process to run on the CPU at a given time.

On a single core machine, the processes execute [_concurrently_](https://en.wikipedia.org/wiki/Concurrency_(computer_science)). That is, the operating system switches between the processes in regular intervals. For example, process `D` executes for a limited time, then its state is saved somewhere and the OS schedules process `B` to execute for a limited time, and so on. This happens back and forth until all the tasks have been finished. From the output, it might look like each process has run to completion, but in reality, the OS scheduler is constantly switching between them.

On a multi-core system—assuming you have four cores—the OS schedules each process to execute on each core at the same time. This is known as [_parallelism_](https://en.wikipedia.org/wiki/Parallel_computing). However, if you create four more processes (bringing the total to eight), each core will execute two processes concurrently until they are finished.

### Threads

Threads are like processes: they have their own instruction pointer and can execute one JavaScript task at a time. Unlike processes, threads do not have their own memory. Instead, they reside within a process’s memory. When you create a process, it can have multiple threads created with the `worker_threads` module executing JavaScript code in parallel. Furthermore, threads can communicate with one another through message passing or sharing data in the process’s memory. This makes them lightweight in comparison to processes, since spawning a thread does not ask for more memory from the operating system.

When it comes to the execution of threads, they have similar behavior to that of processes. If you have multiple threads running on a single core system, the operating system will switch between them in regular intervals, giving each thread a chance to execute directly on the single CPU. On a multi-core system, the OS schedules the threads across all cores and executes the JavaScript code at the same time. If you end up creating more threads than there are cores available, each core will execute multiple threads concurrently.

With that, press `ENTER`, then stop all the currently running Node processes with the `kill` command:

```
sudo kill -9 `pgrep node`
```

`pgrep` returns the process ID’s of all the four Node processes to the `kill` command. The `-9` option instructs `kill` to send a [SIGKILL signal](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html).

When you run the command, you will see output similar to the following:

```
Output[1]   Killed                  node process.js A
[2]   Killed                  node process.js B
[3]   Killed                  node process.js C
[4]   Killed                  node process.js D
```

Sometimes the output might be delayed and show up when you run another command later.

Now that you know the difference between a process and a thread, you’ll work with Node.js hidden threads in the next section.

## Understanding Hidden Threads in Node.js

Node.js does provide extra threads, which is why it’s considered to be multithreaded. In this section, you’ll examine hidden threads in Node.js, which help make I/O operations non-blocking.

As mentioned in the introduction, JavaScript is single-threaded and all the JavaScript code executes in a single thread. This includes your program source code and third-party libraries that you include in your program. When a program makes an I/O operation to read a file or a network request, this blocks the main thread.

However, Node.js implements the `libuv` library, which provides four extra threads to a Node.js process. With these threads, the I/O operations are handled separately and when they are finished, the event loop adds the callback associated with the I/O task in a microtask queue. When the call stack in the main thread is clear, the callback is pushed on the call stack and then it executes. To make this clear, the callback associated with the given I/O task does not execute in parallel; however, the task itself of reading a file or a network request happens in parallel with the help of the threads. Once the I/O task finishes, the callback runs in the main thread.

In addition to these four threads, the [V8 engine](https://v8.dev/), also provides two threads for handling things like automatic garbage collection. This brings the total number of threads in a process to seven: one main thread, four Node.js threads, and two V8 threads.

To confirm that every Node.js process has seven threads, run the `process.js` file again and put it in the background:

```
node process.js A &
```

The terminal will log the process ID, as well as output from the program:

```
Output[1] 9933

A: 2000
A: 4000
```

Note the process ID somewhere and press `ENTER` so that you can use the prompt again.

To see the threads, run the `top` command and pass it the process ID displayed in the output:

```
top -H -p 9933
```

`-H` instructs `top` to display threads in a process. The `-p` flag instructs `top` to monitor only the activity in the given process ID.

When you run the command, your output will look similar to the following:

```
Outputtop - 09:21:11 up 15:00,  1 user,  load average: 0.99, 0.60, 0.26
Threads:   7 total,   1 running,   6 sleeping,   0 stopped,   0 zombie
%Cpu(s): 24.8 us,  0.3 sy,  0.0 ni, 75.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7951.2 total,   6756.1 free,    248.4 used,    946.7 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   7457.4 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   9933 node-us+  20   0  597936  51864  33956 R  99.9   0.6   4:19.64 node
   9934 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.00 node
   9935 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.84 node
   9936 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.83 node
   9937 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.93 node
   9938 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.83 node
   9939 node-us+  20   0  597936  51864  33956 S   0.0   0.6   0:00.00 node
```

As you can see in the output, the Node.js process has seven threads in total: one main thread for executing JavaScript, four Node.js threads, and two V8 threads.

As discussed previously, the four Node.js threads are used for I/O operations to make them non-blocking. They work well for that task, and creating threads yourself for I/O operations may even worsen your application performance. The same cannot be said about CPU-bound tasks. A CPU-bound task does not make use of any extra threads available in the process and blocks the main thread.

Now press `q` to exit `top` and stop the Node process with the following command:

```
kill -9 9933
```

Now that you know about the threads in a Node.js process, you will write a CPU-bound task in the next section and observe how it affects the main thread.

# Более подробно

[Source](https://habr.com/ru/post/479062/)

### Как устроена Node.js. Возможности асинхрона

Давайте посмотрим на этот код: он отлично демонстрирует синхронность выполнения кода в Node.js. Делается запрос на GitHub, затем записываем данные в файл и выводим результат в консоли. Что понятно из этого синхронного кода?

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/843/899/f52/843899f5254f70d7f0f1c870eceba370.png)

Предположим, что это абстрактный web-сервер, который по какому-то роутеру делает операции. Если по данному роутеру приходит входящий запрос, мы делаем дальше запрос, читаем файл, выводим в консоль. Соответственно, то время, которое тратится на запрос и чтение файла, сервер будет заблокирован, никакие другие входящие запросы он обрабатывать не сможет, выполнять другие операции — тоже.

Какие есть варианты решения данной проблемы?

1.  Многопоточность
2.  Неблокирующий ввод/вывод

Для первого варианта (многопоточность) есть хороший пример с web-сервером Apache vs Nginx.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/402/2f4/5ae/4022f45ae819e42fa78eea3bec5847c2.png)

Раньше Apache под каждый входящий запрос поднимал поток: сколько было запросов, столько же было и потоков. В это время Nginx имел преимущество, т. к. использовал неблокирующий ввод/вывод. Здесь вы можете видеть, что с увеличением количества входящих запросов увеличивается объём потребляемой памяти именно у Apache, а на следующем слайде количество обрабатываемых запросов в секунду с количеством подключений у Nginx выше.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/481/38c/0bc/48138c0bc534ea1d904fdc04dd28e9dc.png)

**Наглядно показано, что неблокирующий ввод/вывод эффективнее.**

Неблокирующий ввод/вывод стал возможным благодаря современным операционным системам, которые предоставляют данный механизм — демультиплексор событий.

  

Демультиплексор — это механизм, который принимает от приложения запрос, регистрирует его и выполняет.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/5e8/554/833/5e85548338454bb768be1e1a648d4ac7.png)

В верхней части схемы видно, что у нас есть приложение и в нём выполняются операции (пусть это будет чтение файла). Для этого делается запрос в демультиплексор событий, сюда отправляется ресурс (ссылка на файл), нужная операция и callback. Демультиплексор событий регистрирует этот запрос и возвращает управление непосредственно приложению — таким образом, оно не блокируется. Затем он выполняет операции над файлом, и после этого, когда файл будет прочитан, callback регистрируется в очереди на выполнение. Затем Event Loop постепенно синхронно обрабатывает каждый callback из этой очереди. И, соответственно, возвращает результат приложению. Дальше (если необходимо) всё делается снова.

**Таким образом, благодаря данному неблокирующему вводу/выводу Node.js может быть асинхронным.  
**

Уточню, что в данном случае неблокирующий ввод/вывод предоставляет нам именно операционная система. К неблокирующему вводу/выводу (вообще в принципе к операциям ввода/вывода) мы относим сетевые запросы и работу с файлами.

Это общая концепция неблокирующего ввода/вывода. Когда такая возможность появилась, Райан Дал (Ryan Dahl) — разработчик Node.js — был вдохновлён опытом Nginx, которая использовала неблокирующий ввод/вывод, и решил создать платформу именно для разработчиков. Первое, что ему нужно было сделать, — «подружить» свою платформу с демультиплексором событий. Проблема была в том, что в каждой операционной системе демультиплексор реализован по-разному, и ему пришлось написать обёртку, которая впоследствии стала называться libuv. Это библиотека, написанная на C. Она предоставляет единый интерфейс работы с этими демультиплексорами событий.

### Особенности libuv-библиотеки

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/6a8/ace/421/6a8ace421f0670a007e46e5826c08eaf.png)

В Linux, в принципе, на текущий момент все операции с локальными файлами — блокирующие. Т. е. вроде бы как есть неблокирующий ввод/вывод, но именно при работе с локальными файлами операция всё равно блокирующая. Именно поэтому для эмуляции неблокирующего ввода/вывода libuv использует внутри потоки. Из коробки поднимается 4 потока, и здесь нужно сделать самый важный вывод: если мы выполняем какие-то 4 тяжёлые операции над локальными файлами, соответственно, мы заблокируем всё наше приложение (именно в ОС Linux, в других ОС такого нет).

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/577/943/ae1/577943ae11c7040ebc92adf2cf7971fe.png)

На этом слайде мы видим архитектуру Node.js. Для взаимодействия с операционной системой используется библиотека libuv, написанная на C; для компиляции кода JavaScript'a в машинный код используется движок Google V8, также есть Node.js Core library, где собраны модули для работы с сетевыми запросами, файловой системой и модуль для логирования. Чтобы всё это друг с другом взаимодействовало, написаны Node.js Bindings. Эти 4 компонента составляют саму структуру Node.js. Сам механизм [[Event Loop]]'a находится в libuv.

### Многопоточность — модуль worker_threads

Многопоточность появилась в Node.js благодаря модулю worker_threads в версии 10.5. И в 10-й версии она запускалась исключительно с ключом --experimental-worker, а с 11-й версии стал возможным запуск без него.

Ещё в Node.js есть модуль cluster, но он не поднимает потоки — он поднимает ещё несколько процессов. Масштабируемость приложения — его основная цель.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/f0c/ebc/830/f0cebc8301bcf4d950ff8b4f5ceb7c57.png)

Как вообще выглядит 1 процесс:  
1 процесс Node.js, 1 поток, 1 Event Loop, 1 движок V8 и libuv.

Если мы запускаем X потоков, то у нас это выглядит так:  
1 процесс Node.js, X потоков, Х Event Loop'ов, X движков V8 и X libuv.

#### Схематично это выглядит следующим образом

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/d1b/643/876/d1b643876f7f0540e760dfebc8c741c9.png)

### Давайте разберём пример.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/4e0/b65/ebb/4e0b65ebb7220d0a070d734e49c95abd.png)

Простейший web-сервер на Express'е. Есть 2 route'а — / и /fat-operation.

Также есть функция generateRandomArr(). Она наполняет массив двумя миллионами записей и сортирует его. Запустим сервер.

Делаем запрос на /fat-operation. И в тот момент, когда выполняется операция сортировки массива, отправляем ещё один запрос на route /, но для получения ответа нам приходится ждать до тех пор, пока не выполнится сортировка массива. Это классическая реализация через один поток. Теперь подключаем модуль worker_threads.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/a33/835/066/a338350669ec8b293972f133b2b7d874.png)

Делаем запрос на /fat-operation и следом — на /, от которого тут же получаем ответ — Hello world!

Для операции сортировки массива мы подняли отдельный поток, у которого есть свой экземпляр Event Loop, и он никак не влияет на выполнение кода в основном потоке.

Поток будет «уничтожен», когда у него не будет операций для выполнения.

Смотрим исходный код. Регистрируем worker в строке 26 и, если нужно, передаём ему данные. В данном случае я ничего не передаю. И затем подписываемся на события: на ошибку и на месседж. В самом worker'е происходит вызов функции, массив из двух миллионов записей сортируется. Как только он отсортировался, мы через post_message отправляем результат в основной поток ok.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/97f/dd2/e42/97fdd2e4294db9e3a3e9548db3fc3c8f.png)

В основном потоке мы ловим это сообщение и отправляем результат finish. У worker'а и основного потока общая память, поэтому мы имеем доступ к глобальным переменным всего процесса. Когда мы передаём данные из основного потока в worker, worker получает только копию.

Основной поток и поток worker'а мы можем описать в одном файле. Модуль worker_threads предоставляет API, благодаря которому мы можем определить, в каком потоке сейчас выполняется код.

![image](https://habrastorage.org/r/w1560/getpro/habr/post_images/939/035/ad7/939035ad7f869b719b8654b58b8248c7.png)

#### Дополнительная информация

Делюсь ссылками на полезные ресурсы и ссылкой на презентацию Райана Дала, когда он презентовал Event Loop (интересно посмотреть).

Event Loop

1.  [Перевод статьи из документации Node.js](https://medium.com/devschacht/event-loop-timers-and-nexttick-18579cd122e0)
2.  [https://blog.risingstack.com/node-js-at-scale-understanding-node-js-event-loop/](https://blog.risingstack.com/node-js-at-scale-understanding-node-js-event-loop/)
3.  [https://habr.com/ru/post/336498/](https://habr.com/ru/post/336498/)

Worker_threads

1.  [https://nodejs.org/api/worker_threads.html#worker_threads_worker_workerdata](https://nodejs.org/api/worker_threads.html#worker_threads_worker_workerdata) — API
2.  [https://habr.com/ru/company/ruvds/blog/415659/](https://habr.com/ru/company/ruvds/blog/415659/)
3.  [https://nodesource.com/blog/worker-threads-nodejs/  
    ](https://nodesource.com/blog/worker-threads-nodejs/)
4.  [Original slides from Ryan Dahl's presentation (through VPN)](https://www.slideshare.net/AartiParikh/original-slides-from-ryan-dahls-nodejs-intro-talk)