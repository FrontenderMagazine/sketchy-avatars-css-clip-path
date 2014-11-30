# Аватары с рваными краями при помощи CSS clip-path

Примечание от Криса Коера, автора блога, в котором была размещена статья:


*Следующий текст является гостевой публикацией от [Райана Шерфа (Ryan Scherf)][1].
Райан нашел изящный способ придать аватарам разного вида грубые, неровные
края. Как будто их обрезал ножницами кто-то, не шибко владеющий ножницами. Что
замечательно, это естественная техника прогрессивного  улучшения, и выполняется
она на простом CSS.*

Работая с таким креативным и веселым брэндом, как [Quirky][2], мы постоянно
обдумываем возможности перенести его атмосферу в веб. Повсюду на сайте можно
встретить элементы в рисованном стиле. Очень сложно добиться этого ощущения
нарисованности без использования множества изображений. С помощью небольшого
количества легкой тригонометрии и очень базовых знаний CSS-свойства `clip-path`,
мы сможем достигнуть этого с относительной простотой и высокой
производительностью.

![Вот что мы будем делать][Аватары с рваными краями]Вот что мы будем делать. Обратите внимание на то, 
что края неровные и различаются в каждом случае.

## Почему мы не используем маску?

Для примера, вот маска, созданная в SVG:

    img {
      mask: url(mask.svg) top left / cover;
    }

Свойство `mask` может ссылаться на внешний файл SVG или по id на SVG, записанный
в документе.

Но что, если вы хотите задать уникальную форму для каждого отдельно взятого
аватара, а не применять одну и ту же ко всем? Мы могли бы программно
сгенерировать множество различных SVG-фигур для использования. Но мы можем добиться
того же самого, применив те же математические вычисления для генерации `clip-
path` с помощью (S)CSS.

## Что с поддержкой браузерами?

Поддержка браузерами свойства `clip-path`, при использовании фигур вроде
`polygon()`, начинается с Chrome 24+, Safari 7+, Opera 25+, iOS 7.1+, Android
4.4+. В Firefox `clip-path` реализован только с путями, объявленными через SVG
(мы учтем это).  До сих пор не поддерживается в IE.

Вам придется писать `-webkit-clip-path`, так как это единственное, что
поддерживается прямо сейчас, но наверное лучше дописать туда `clip-path` тоже.
Если IE или Firefox начнут поддерживать это свойство, оно вероятнее всего будет
беспрефиксным.

## Обрезание контуров в двух словах

Существует несколько фигур, которые можно использовать для CSS-маски, но в нашем
случае `polygon` подходит больше всего, так как дает больше точек и гибкости для
создания нашего рисованного эффекта.

Вы передаете в `polygon()` список из значений X и Y для вершин, вот так: `<x0>
<y0>, <x1> <y1>, … <xn> <yn>`. Это отрисует фигуру по точкам **в заданном
порядке** и обрежет любой контент **за пределами**  только что созданной формы.

    /* 
      Это создаст шестигранник, в котором отсчет
      начинается с самой верхней точки */
    
    .hexagon {
      clip-path: polygon(50% 0, 100% 25%, 100% 75%, 50% 100%, 0 75%, 0 25%);
    }

Вот простой пример в действии:

[Простой шестигранник][3]

## Не столь страшна математика

Наш шестигранник довольно клевый, но все же он не добавит нам реального эффекта
нарисованности. Он выглядит грубо — слишком мало граней. Лучшим способом
представить нарисованную от руки фигуру является последовательность из коротких
линий между двумя точками. Чем больше точек мы имеем,  тем больше коротких
отрезков мы создаем. По факту, с достаточным количеством точек, мы можем создать
`polygon` со столь гладкой формой, что он начнет походить на `circle`.

Вот пример, использующий 200 точек:

[Окружность через полигон][4]

### Откуда возьмутся точки?

Вот здесь появляется еще немного математики. Возможно, вы проходили
тригонометрию в старших классах? К одной из фундаментальных идей этой науки
относится  понятие **единичной окружности**. В сущности, она предоставляет
формулу (использующую число Пи), с помощью которой можно сгенерировать любое
количество точек на окружности.

![Единичная окружность][Единичная окружность]Единичная окружность (из [Wikipedia][5])

Если мы соединим наши точки, мы получим фигуру, выглядящую так:

![Соединяем точки!][Соединяем точки на единичной окружности]Соединяем точки!

Все еще грубо, но немного больше походит на начерченное от руки.

### Больше точек!

Мы знаем как делать шестигранники и окружности с помощью `clip-path: polygon()`,
но как же мы заставим их выглядеть нарисованными?

*   Отрегулируем количество вершин (чем больше их, тем меньше длина сегментов)
*   Добавим немного смещений по X и Y (так сегменты станут неоднородными)

Давайте занесем это в SCSS и создадим функцию, которая сделает грязную работу за
нас.

Мы задействуем:

* `random()`
* `cos()`
* `sin()`

Наиболее подходящие нам математические функции:

     /* 
      Генерируем произвольные точки на 
      единичной окружности под углом t 
     */
      
    $x: cos(t);
    $y: sin(t);

И запишем это в нужном синтаксисе вот так:

    $w: 160px    // Ширина аватара
    $n: 60;      // Количество точек на окружности
    
    @function sketchAvatar() {
      $points: ();
    
      @for $i from 0 through $n {
        $points: append($points, ($w / 2) * (1 + cos((2 * pi() * $i / $n))) ($w / 2) * (1 + sin((2 * pi() * $i / $n))), comma);
      } 
      
      @return $points;
    }

Это немного топорно. Произойдет то, что, начиная с верхней точки посередине
нашей фигуры, мы сгенерируем набор вершин по окружности для 60 равных
промежутков.

## Собираем все вместе в различных вариациях

Код  описанный выше, просто производит довольно гладкие и однородные полигоны,
так что нам надо добавить расхождения. Все что нам нужно, это просто сместить
наши точки  в различных направлениях для придания того самого эффекта, который
мы хотим. Числа в переменных `$lower` и  `$upper` могут быть какими угодно, в
зависимости от того, чего вы желаете достигнуть.

    $w:     120px;   // Overall width
    
    @function sketchAvatar() {
      $n: 	  60;     // Number of points
      $lower: -80;    // Lower variance
      $upper: 80;     // Upper variance
    
      $points: ();
    
      @for $i from 0 through $n {
        $points: append($points, ($w / 2) * (1 + cos((2 * pi() * $i / $n))) + (rand($lower, $upper) / 100) ($w / 2) * (1 + sin((2 * pi() * $i / $n))), comma);
      } 
      
      @return $points;
    }

Мы сделали это! Эскизные, уникальные аватары через CSS-свойство `clip-path:
polygon()`:

[Эскизные аватары][6]

## Заставляем это работать в Firefox

Крис здесь! Я подумал, раз Firefox не поддерживает это описанным образом, но
поддерживает синтаксис SVG, мы можем сделать нечто вроде полифилла.

    .avatar {
      clip-path: polygon( … ) /* Firefox: не-а */
      clip-path: url(#clip); /* Firefox: да */
    }

Так для каждого аватара, я…

1.  Выгружаю вершины полигона в свойство `content` псевдо-элемента (для
элемента, который поддерживает валидные псевдо-элементы, например, для родительского `div`)
2.  Извлекаю эти значения через JavaScript
3.  Привожу значения в соответствие формату SVG (например, без "px")
4.  Вставляю новый `<svg>` с путем через `<clipPath>` готовым к использованию

&nbsp;

    $(".user").each(function(i) {
     
      var path = window.getComputedStyle(this, ':after').getPropertyValue('content');
      
      // делаем уборку
      svgPolygonPoints = 
        path
          .replace(/px/g, "")
          .replace(/polygon/, "")
          .replace(/\(/, "")
          .replace(/\)/, "")
          .replace(/\;/g, "")
          .replace(/"/g, "")
          .replace(/\'/g, "");
        
      // Чтобы заставить это действительно работать, нужно создать <div>
         и поместить это внутри, смотрите ниже.
      var svg = $("<svg width='0' height='0'>")
        .append("<defs><clipPath id='clip-" + (i+1) +"'><polygon points='" + svgPolygonPoints +"' /></clipPath></defs>");
      
      $("body").append(svg);
        
    });


*Примечание переводчика: тут Крис обнаруживает, что этот код не работает, но
если вставить результат в новый файл HTML, картинка отрисовывается. В ситуации
помогает разобраться  девушка под ником Amelia BR, и Крис дает ссылку на ее
комментарий. Ниже будет приведен перевод данного комментария.*

Оно не работает! Ха-ха. Прочитайте [ответ Амелии][7]:

> Это не срабатывает, потому что вы используете JQuery. JQuery не понимает, как
создавать элементы из пространства имен SVG, или элементы, которые могут
находится внутри SVG. Это значит что она создаст группу элементов, которые будут
выглядеть нормально в инспекторе DOM — они имеют корректные имена тегов — но
фактически все они будут `HTMLUnknownElement` в DOM. И так вы не сможете
использовать их в качестве содержимого SVG.

> С другой стороны, когда вы копируете эту разметку в отдельный файл, этот файл
будет прочитан парсером HTML5, который может распознать все элементы `<svg>` и
их потомков как, ну, `SVGElements`.

> У вас есть 2 варианта:

> 1. Использовать ванильный javascript для вставки SVG с помощью
`document.createElementNS("www.w3.org/2000/svg", "svg")` (и то же самое для
потомков). 

> 2. Использовать парсер HTML5 самостоятельно через (a) создание подставного
элемента `div`, (b) временная вставка в него полученного SVG, в свойство
`.innerHTML`, или через jQuery-метод `.append()`, (c) выгрузка SVG из контейнера
`div`.

> Здесь я использовала 2 вариант. Это заставило `clip-path` работать в Firefox:

>[Codepen Амелии][8]

Это было таким:

    .user:nth-child(1)  {
      clip-path: polygon(120.04px 60px …);
    }

стало:

    <svg width="0" height="0">
      <defs>
        <clippath id="clip-1">
          <polygon points="120.04 60, … "></polygon>
        </clippath>
      </defs>
    </svg>


 [1]: http://twitter.com/ryanscherf
 [2]: http://quirky.com
 [3]: http://codepen.io/rscherf/pen/haoEk
 [4]: http://codepen.io/rscherf/pen/zpuBg
 [5]: http://en.wikipedia.org/wiki/Unit_circle
 [6]: http://codepen.io/rscherf/pen/pzgKt
 [7]: http://css-tricks.com/sketchy-avatars-css-clip-path/#comment-1586734
 [8]: http://codepen.io/AmeliaBR/pen/qEBxXW

[Аватары с рваными краями]: img/sketchy-avatars.png
[Единичная окружность]: img/unit-cricle.png
[Соединяем точки на единичной окружности]: img/path.png