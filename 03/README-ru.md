## Uniform-переменные

В предыдущей главе мы увидели каким образом GPU управляет большим количеством параллельных потоков, каждый из которых отвечает за назначение цвета небольшой части изображения. Каждый из параллельных потоков не знает о состоянии остальных, но всё же нам бывает нужно передавать входные данные от CPU ко всем потокам одновременно. Из-за особенностей архитектуры графических карт, все такие входные данные будут одинаковыми для всех потоков (однородными, *uniform*) и доступными *только для чтения*. Другими словами, каждый поток принимает на вход одни и те же данные, которые он может прочитать и не может перезаписать.

Эти входы называются однородными (`uniform`) и могут иметь практически любой из поддерживаемых типов: `float`, `vec2`, `vec3`, `vec4`, `mat2`, `mat3`, `mat4`, `sampler2D` и `samplerCube`. Uniform-переменные с указанием своих типов объявляются вначале шейдера сразу после указания точности по умолчанию.

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;  // Размер изображения (ширина, высота)
uniform vec2 u_mouse;       // Положение курсора мыши в пикселях
uniform float u_time;       // Время в секундах с момента загрузки
```

Uniform-переменные можно представлять себе как маленькие мостики между CPU и GPU. Имена могут изменяться от примера к примеру, но в этой серии примеров я всегда передаю следующее: `u_time` (время в секундах с момента запуска шейдера), `u_resolution` (размер изображения) и `u_mouse` (положение мыши на изображении, выраженное в пикселях). Я буду писать `u_` вначале имён uniform-переменных, чтобы их происхождение было обозначено явно, но на практике вы столкнётесь с самыми разными именами uniform-переменных. Например, [ShaderToy.com](https://www.shadertoy.com/) объявляет эти же переменные со следующими именами:

```glsl
uniform vec3 iResolution;   // разрешение области изображения (в пикселях)
uniform vec4 iMouse;        // координаты мыши в пикселях. xy - текущие, zw - клик
uniform float iGlobalTime;  // время работы шейдера (секунды)
```

Хватит слов, давайте посмотрим на юниформы в действии. В следующем коде мы используем `u_time` (количество секунд с момента запуска шейдера) в комбинации с функцией синуса чтобы анимировать изменение количества красного цвета на экране.

<div class="codeAndCanvas" data="time.frag"></div>

Как видите, GLSL содержит ещё много сюрпризов. GPU аппаратно ускоряет угловые, тригонометрические и экспоненциальные функции. Вот некоторые из них: [`sin()`](../glossary/?search=sin), [`cos()`](../glossary/?search=cos), [`tan()`](../glossary/?search=tan), [`asin()`](../glossary/?search=asin), [`acos()`](../glossary/?search=acos), [`atan()`](../glossary/?search=atan), [`pow()`](../glossary/?search=pow), [`exp()`](../glossary/?search=exp), [`log()`](../glossary/?search=log), [`sqrt()`](../glossary/?search=sqrt), [`abs()`](../glossary/?search=abs), [`sign()`](../glossary/?search=sign), [`floor()`](../glossary/?search=floor), [`ceil()`](../glossary/?search=ceil), [`fract()`](../glossary/?search=fract), [`mod()`](../glossary/?search=mod), [`min()`](../glossary/?search=min), [`max()`](../glossary/?search=max) и [`clamp()`](../glossary/?search=clamp).

Настало время поиграть с кодом выше.

* Уменьшите частоту так, чтобы изменение цвета стало почти незаметным.

* Увеличивайте её до тех пор, пока не увидите сплошной цвет без мерцания.

* Поиграйтесь с каналами RGB на различных частотах, чтобы добиться какого-нибудь интересного поведения.

## gl_FragCoord

Подобно тому, как GLSL объявляет выходное значение `vec4 gl_FragColor` по умолчанию, он так же предоставляет вход `vec4 gl_FragCoord`, содержащий координаты *пикселя* или *фрагмента экрана*, над которым работает данный поток. С помощью `vec4 gl_FragCoord` мы можем узнать где именно поток работает внутри изображения. В данном случае мы не называем входное значение однородным, потому что оно меняется от потока к потоку и называется изменяющимся (*varying*).

<div class="codeAndCanvas" data="space.frag"></div>

В коде выше мы нормализуем координаты фрагмента, разделяя их на разрешение изображения. В результате значения переходят в диапазон между `0.0` и `1.0`, что упрощает отображение значений X и Y на красный и зелёный каналы.

В мире шейдеров у нас нет нормальных отладочных инструментов, поэтому приходится задавать переменным яркие цвета и пытаться извлечь из них смысл. Вы увидите, что иногда программирование на GLSL похоже на засовывание корабля в бутылку. Оно столь же сложно, сколь красиво и захватывающе.

![](08.png)

Настало время проверить наше понимание приведённого выше кода.

* Укажите где находятся координаты `(0.0, 0.0)` на изображении.

* Как насчёт `(1.0, 0.0)`, `(0.0, 1.0)`, `(0.5, 0.5)` и `(1.0, 1.0)`?

* Догадайтесь как использовать `u_mouse`, зная, что координаты даны в пикселях и НЕ нормализованы. Можете ли вы изменять цвета с помощью этой переменной?

* Придумайте какой-нибудь интересный способ изменения цветов с помощью `u_time` и `u_mouse`.

После выполнения этих упражнения у вас скорее всего возникнет вопрос: где ещё можно применить мощь шейдеров? В следующей главе вы научитесь создавать шейдерные инструменты на three.js, Processing, и openFrameworks.
