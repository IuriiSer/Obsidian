 **_Принцип разделения интерфейсов_**

Этот принцип был сформулирован Робертом Мартином, когда он консультировал Xerox, и он очевиден.  

> Объекты не должны зависеть от интерфейсов, которые они не используют
 
ПО должно разделяться на независимые части. Побочные эффекты необходимо сводить к минимуму, чтобы обеспечивать независимость.  
  
Убедитесь, что вы не заставляете объекты реализовывать методы, которые им никогда не понадобятся. Вот пример:   

```
interface Animal {
  eat: () => void;
  walk: () => void;
  fly: () => void;
  swim: () => void;
}
```

Не все животные могут fly, walk или swim, поэтому эти методы не должны быть частью интерфейса или должны быть необязательными.