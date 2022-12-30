React Fiber — прогрессивная реализация ключевого алгоритма React. Это кульминационное достижение двухгодичных исследований команды разработчиков React.

Цель Fiber в увеличении производительности при разработке таких задач как анимация, организация элементов на странице и движение элементов. Ее **главная особенность** это **инкрементный рендеринг**: **способность разделять работу рендера на единицы и распределять их между множественными фреймами.**

## Ключевые понятия:

-   В пользовательских интерфейсах не важно чтоб каждое обновление было применено сразу; фактически такое поведение будет лишним, оно будет способствовать падению фреймов и ухудшению UX.
-   Разные типы апдейтов имеют разные приоритеты – обновления анимации должны заканчиваться быстрее чем, скажем, обновление данных хранилища.
-   Проталкивающий (push-based) подход трубует от приложения (вас, разработчика) решать как планировать работу. Протягивающий (pull-based) подход позволяет фреймворку принимать решения за вас.

Реакт на данный момент не имеет приемущества планирования в значитильной мере; результаты обновлений всего поддерева будут отрисовываться незамедлительно. Тщательный отбор елементов в алгоритме ядра React, чтобы применить планирование – ключевая идея Fiber.

## Что же такое Fiber?

Мы будем обсуждать сердце архитектуры React Fiber. **Fiber — это более низкоуровневая абстракция над приложением** чем разработчики привыкли считать.

Главной цели архитектуры Fiber — позволить React воспользоваться планированием. Конкретно, нам нужно иметь возможность:

-   остановить работу и вернуться к ней позже.
-   приоритизировать разные типы работы.
-   переиспользовать работу проделанную ранее.
-   отменить работу, если она больше не нужна.