**_Принцип инверсии зависимостей_**

Этот принцип невозможно переоценить. Мы должны полагаться на абстракции, а не на конкретные реализации. Компоненты ПО должны иметь низкую связность и высокую согласованность.  
  
Заботиться нужно не о том, как что-то устроено, а о том, как оно работает. Простой пример – использование дат в JavaScript. Вы можете написать для них свой слой абстракции. Тогда если у вас сменится источник получения дат, вам нужно будет внести изменения в одном месте, а не в тысяче.  
  
Иногда добавление этого уровня абстракции требует усилий, но в конечном итоге они окупаются.  
  
В качестве примера взгляните на [date-io](https://github.com/dmtrKovalenko/date-io), в этой библиотеке создан тот уровень абстракции, который позволяет вам использовать её с разными источниками дат.