---
title: Есть ли ООП в Rust?
author: Михаил Панков
categories: обучение
---

{% img '2017-04-12-communicating-intent/teaser.png' alt:'teaser' %}

Многие программисты уже умеют программировать на объектно-ориентированных
языках. Rust не является классическим объектно-ориентированным языком, но
основные инструменты ООП можно применять и в нём.

В этой статье мы рассмотрим, как программировать на Rust в ООП-стиле. Мы будем
делать это на примере: построим иерархию классов в учебной задаче.

Наша задача - это работа с геометрическими фигурами. Мы будем выводить их на
экран в текстовом виде и вычислять их площадь. Наш набор фигур - прямоугольник,
квадрат, эллипс, круг.

Теперь определимся с тем, что значит "программировать в ООП-стиле". Всё ООП в
одной статье охватить невмозжно, поэтому выделим несколько основных черт,
которые будем пытаться реализовать.

### Абстракция данных

Объединение данных одной сущности в одном объекте. Это возможность представить
сущность сразу совокупностью её свойств, а не отдельными переменными.

Структуры в Си, записи в Haskell, объекты в Java - все они реализуют абстракцию
данных.

В Rust абстракция данных представлена структурами.

### Инкапсуляция

Объединение данных и методов работы с этими данными. Это возможность вызвать у
экземпляра объекта метод, который делает что-то с экземпляром. Это значит, что
вызов метода автоматически передаёт объект в качестве контекста в метод - часто
в виде первого аргумента. Это также значит, что объект или класс предоставляет
своё пространство имён, в котором находятся его методы.

Инкапсуляция есть в C++, Java, и многих других объектно-ориентированных языках.

В Rust есть инкапсуляция, работает через методы, определённые на структурах.

### Сокрытие

Сокрытие деталей реализации функциональности от пользователей. Это значит, что
есть как минимум общие и частные поля и методы. Общие методы доступны
пользователям объекта, а частные доступны только изнутри реализации.

Сокрытие есть в C++, Java и многих других объектно-ориентированных языках.

В Rust сокрытие также есть - существуют общие и частные поля структур и методы.
Частные элементы доступны в реализации функциональности, но недоступны снаружи.

### Наследование

Переиспользование методов классов-родителей, т.е. переиспользование более общих
реализаций в более частных. Это значит, что метод родителя будет вызван в
отсутствие реализации метода для потомка.

Наследование есть в C++, Java и многих других объектно-ориентированных языках.

В Rust нет классического наследования, но есть возможность изобразить его с
помощью типажей.

### Полиморфизм подтипов

Возможность работать с объектами-потомками так же, как с объектами-родителями.
Т.е. если все потомки наследуются от одного класса, мы можем обработать их все
как экземпляры этого класса.

Полиморфизм подтипов есть в C++, Java и многих других объектно-ориентированных
языках.

В Rust есть полиморфизм подтипов и реализуется он через типажи и типажи-объекты.

## Самая наивная реализация

Давайте начнём с реализации структур прямоугольника и квадрата.

``` rust
  // прямоугольник
  struct Rectangle {
      // ширина
      width: f64,
      // длина
      length: f64,
  }

  // квадрат
  struct Square {
      // сторона
      side: f64,
  }
```

Это определения структур. Можно сказать, что это подобие классов. С этими
определениями мы сможем создавать экземпляры прямоугольников и квадратов.

Ещё нам нужно определить методы подсчёта площади для обеих структур.

``` rust
  impl Rectangle {
      fn area(&self) -> f64
      {
          self.width * self.length
      }
  }
  impl Square {
      fn area(&self) -> f64
      {
          self.side * self.side
      }
  }
```

Этим всем можно пользоваться так:

``` rust
fn main() {
    let rect1 = Rectangle { width: 3., length: 5. };
    let rect2 = Rectangle { width: 4., length: 6. };

    let sq1 = Square { side: 8. };
    let sq2 = Square { side: 4. };

    let rects = [&rect1, &rect2];
    let squares = [&sq1, &sq2];

    for r in rects.iter() {
        println!("Площадь равна {}", r.area());
    }

    for s in squares.iter() {
        println!("Площадь равна {}", s.area());
    }
}
```

Код выше выведет такие сообщения:

``` text
Площадь равна 15
Площадь равна 24
Площадь равна 64
Площадь равна 16
```

В строках 2, 3 и 5, 6 мы создаём экземпляры прямоугольников и квадратов,
используя синтаксис литералов структур.

В строках 8, 9 мы складываем прямоугольники и квадраты в соответсвующие массивы,
а затем обходим их, вычисляем и печатаем площадь.

Сейчас мы не печатаем саму фигуру. Чтобы печатать её, нужно реализовать ещё один
метод. Этот метод должен делать форматированный вывод. Идиоматично это делается
с помощью типажа Display. Однако мы ещё не говорили про типажи, поэтому давайте
отложим этот вопрос.

### Оценка

- Хорошо
  - Можно написать не глядя
- Плохо
  - Новый тип --- новый код
  - Нет наследования кода
  - Нет обобщённой обработки
  - Нет защиты деталей реализации
  - Нет проверки данных при создании экземпляров

Давайте решать эти проблемы. Начнём с самой простой --- с проверки данных при
создании экземпляров.

## Проверка данных при создании экземпляров

Проверку значений полей при создании экземпляров можно реализовать с помощью
метода-конструктора.

В Rust конструкторы не отличаются от других методов синтаксически. Однако обычно
простейший метод-конструктор называется `new`.

Более сложные констукторы реализуются с помощью паттерна Builder и чаще всего
называются вроде `with_proxy_config` (констуктор HTTP-клиента, принимающий
конфигурацию прокси).

Давайте реализуем наши конструкторы.

``` rust
impl Rectangle {
    fn new(width: f64, length: f64) -> Option<Rectangle> {
        if width > 0. && length > 0. {
            Some( Rectangle { length, width } )
        } else {
            None
        }
    }
    // ...
}

impl Square {
    fn new(side: f64) -> Option<Square> {
        if side > 0. {
            Some( Square { side } )
        } else {
            None
        }
    }
    // ...
}
```

Обратите внимание, что наши конструторы возвращают `Option<Rectangle>` и
`Option<Square>`. Это сделано чтобы не возвращать экземпляр в случае
неправильного вызова конструктора. В Rust нет исключений. Мы не создаём квадраты
с отрицательными сторонами. Поэтому мы возвращаем перечисление - если создание
удалось, у нас будет `Some<T>`; если не удалось - `None`. Строго говоря, это не
совсем идиоматичная обработка ошибок. Лучше было бы возвращать `Result<T, E>`,
но это уже совсем другая тема.

На стороне вызывающего нам нужно будет обработать возможную ошибку. Сейчас мы не
будем её обрабатывать, и просто вызовем `.unwrap()` - это даёт `T` из `Some(T)`,
или паникует если `Option` оказался `None`. Паника - это раскрутка стека с
вызовом деструкторов и завершение программы с опциональной распечаткой обратной
трассировки вызовов.

``` rust
fn main() {
    let rect1 = Rectangle::new(3., 5.).unwrap();
    let rect2 = Rectangle::new(4., 6.).unwrap();;
    let rect3 = Rectangle::new(-4., 6.).unwrap();;

    let sq1 = Square::new(8.).unwrap();
    let sq2 = Square::new(4.).unwrap();

    // ...
```

Мы улучшили инкапсуляцию, однако напрямую создать структуру с помощью синтаксиса
литералов всё ещё можно. Давайте теперь разберёмся с этим.

## Сокрытие частных полей

Разделение частных и общих полей и методов работает на уровне модулей. Внутри
модуля все функции, методы и поля структур доступны без ограничений - независимо
от того, являются они частными или общими. Вне модуля частные элементы в общем
случае не доступны.

Модули могут вкладываться друг в друга. В корневом модуле может быть модуль `a`,
в нём модуль `b`, а в нём - модуль `c`. При этом частные элементы вышестоящих
модулей доступны во вложенных модулях. Так, частные элементы корневого модуля
доступны во всех модулях - корневом, `a`, `b` и `c`. Частные элементы модуля `a`
доступны в модулях `b` и `c`, но не доступны в корневом модуле. И так далее.

Зная это, мы можем сокрыть нужные нам элементы, являющиеся деталями реализации.
В данном случае это поля `width` и `length` прямоугольника и `side` квадрата.

``` rust
mod figures {
    pub struct Rectangle {
        width: f64,
        length: f64,
    }

    pub struct Square {
        side: f64,
    }

    impl Rectangle {
        pub fn new(width: f64, length: f64) -> Option<Rectangle> {
            if width > 0. && length > 0. {
                Some( Rectangle { length, width } )
            } else {
                None
            }
        }
        pub fn area(&self) -> f64
        {
            self.width * self.length
        }
    }

    impl Square {
        pub fn new(side: f64) -> Option<Square> {
            if side > 0. {
                Some( Square { side } )
            } else {
                None
            }
        }
        pub fn area(&self) -> f64
        {
            self.side * self.side
        }
    }
}


fn main() {
    let rect1 = figures::Rectangle::new(3., 5.).unwrap();
    let rect2 = figures::Rectangle::new(4., 6.).unwrap();;
    // error: field `width` is private
    let rect3 = figures::Rectangle { width: 3., length: 5. };

    let sq1 = figures::Square::new(8.).unwrap();
    let sq2 = figures::Square::new(4.).unwrap();

    let rects = [&rect1, &rect2];
    let squares = [&sq1, &sq2];

    for r in rects.iter() {
        println!("Площадь равна {}", r.area());
    }

    for s in squares.iter() {
        println!("Площадь равна {}", s.area());
    }
}
```

Теперь создание структур с помощью синтаксиса литералов невозможно. Этого мы и
добивались.

Возникло одно неудобство - к структурам теперь надо обращаться в модуль
`figures`. Это легко решается с помощью импорта имён:

``` rust
use figures::{Rectangle, Square};
```

### Оценка

Теперь наша реализация выглядит так:

- Хорошо
  - Достаточно просто
  - Есть защита деталей реализации
  - Есть проверка данных при создании экземпляров
- Плохо
  - *Новый тип --- новый код*
  - *Нет наследования кода*
  - *Нет обобщённой обработки*

Дальше мы будем решать проблему обобщённой обработки фигур.

## Обобщение через перечисление

Самый простой способ обобщить фигуры - использовать перечисление:

``` rust
    pub enum Figure {
        Rect(Rectangle),
        Sq(Square),
    }

    impl Figure {
        pub fn area(&self) -> f64
        {
            match self {
                &Figure::Rect(ref r) => r.area(),
                &Figure::Sq(ref s) => s.area(),
            }
        }
    }
```

Теперь мы можем хранить их в одном массиве и обрабатывать единообразно:

``` rust
fn main() {
    let rect1 = Figure::Rect(Rectangle::new(3., 5.).unwrap());
    let rect2 = Figure::Rect(Rectangle::new(4., 6.).unwrap());

    let sq1 = Figure::Sq(Square::new(8.).unwrap());
    let sq2 = Figure::Sq(Square::new(4.).unwrap());

    let figures = [&rect1, &rect2, &sq1, &sq2];

    for f in figures.iter() {
        println!("Площадь равна {}", f.area());
    }
}

```

Мы могли бы даже избавиться от ручного оборачивания фигур в соответствующие
варианты перечисления, но не будем этого делать, т.к. у этого способа обобщения
кода много других недостатков.

### Оценка

- Хорошо
  - *Достаточно просто*
  - *Есть защита деталей реализации*
  - *Есть проверка данных при создании экземпляров*
  - Есть обобщённая обработка
- Плохо
  - *Новый тип --- новый код*
  - *Нет наследования кода*
  - Размер объектов максимален

# Простое наследование

## Код

## Оценка

# Типажи

## Структуры точки и квадрата

## Структуры прямоугольника и квадрата

## Реализация типажей

## Создание структур

## Использование

## Оценка

# Наследование через типажи

## Площадь прямоугольников

## Определяем квадрат как прямоугольник

## Добавляем прямоугольникам площадь

## Реализуем наследование

# Инкапсуляция

## Метод создания точки

## Вызов метода

## Доступ к частным полям остаётся

# Сокрытие

## Используем модули

## Частные поля теперь не доступны

# Обобщённая обработка

## Код

## Результат

## Печать самой фигуры

## Обход проблемы

### Через два среза типажей-объектов

### Через объединение всех типажей

## Оценка

# Проблема с обобщением площади

## Добавляем эллиптические фигуры

## Печать расщепляется

## Обобщаем печать ещё раз

# Как выбирать способ "наследования"

# Заключение