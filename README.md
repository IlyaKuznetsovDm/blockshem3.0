
## Распознаватель блок-схем


## Постановка задачи
Разработать модель, которая по данному на вход изображению блок-схемы сгенерирует соответствующий код на языке Python. Модель должна распознавать основные элементы блок-схем, последовательность их расположения, в том числе текст, имеющийся внутри элементов или вблизи них; затем переводить элементы блок-схемы в подходящие языковые команды и собирать рабочую версию кода - прототип функции. Модель рассчитана на простые сценарии, реализуемые в рамках школьной программы. Приложение на основе такой модели может служить тренажером по программированию для формирования логического мышления и навыков реализации простых алгоритмов на языке Python.

## Описание работы
В данной работе мы сосредоточились на хороших данных, построенных в нотации ДРАКОН, чтобы довести построение схемы до логического конца и построить модель, способную распознавать более сложные структуры на схемах.



### Рекомендуемые параметры входного изображения
1.Размер фигуры (иконки) не менее 70х30 пикселей, площадь фигуры от 1000 до 20000 пикселей. Толщина границы фигуры 2 пикселя. Расстояние между двумя соседними фигурами в горизонтальном направлении не менее 20 пикселей, в вертикальном - не менее 40 пикселей.
2.Толщина стрелки задается автоматически (основная стрелка - 3 пикселя, остальные стрелки - 1 пиксель). Длина каждого отрезка основной стрелки не менее 40 пикселей, длина каждого отрезка остальных (более тонких) стрелок не менее 70 пикселей (отрезки между точками фигурами и точками вхождения стрелок также не менее 70 пикселей).
3.Шрифт текста Lato 19 пикселей. Допускается жирный шрифт, например, в названии программы. Форматирование текста внутри фигуры по центру. Расстояние между строками текста внутри одной фигуры не менее 10 пикселей, расстояние между текстом внутри фигуры и ее границами - не менее 10 пикселей. Для слов вне фигуры (Yes/No) расстояние до границ близлежащих фигур не менее 4 пикселей, расстояние до отрезка стрелки - не менее 10 пикселей.
4.Слева и справа от знаков > или < ставится по одному пробелу, слева и справа от знаков = и % пробелы не ставятся.

### Этапы работы программы
- Генерация схем 
- Разработка требований к входному изображению со схемой
- Детектирование текста
- Распознавание текста 
- Детектирование фигур
- Распознавание фигур
- Детектирование отрезков, составляющих стрелки
- Распознавание отрезков
- Распознавание точки вхождения одной стрелки в другую (конца блока кода)
- Модель хранения данных, получаемых после парсинга блок-схемы
- Перенос распаршенных данных в генератор кода
- Генерация кода

## Реализация
- На вход модели подается изображение блок-схемы
- На выходе получается прототип рабочей функции на языке Python 
- Класс ImageHandler служит для парсинга изображения блок-схемы
- Класс DataStructureConnector связывает элементы блок-схемы с соотвестсвующими элементами кода (ключевыми словами и синтаксическими конструкциями)
___
### Парсинг изображения блок-схемы
Перевод изображения блок-схемы в программный код прототипа функции осуществляется следующим образом:
1. Текст. Требуется распознавать текст, находящийся внутри геометрических фигур (латинские буквы, символы математичсеких/логических операций, скобки) или рядом со стрелками (слова Да/Нет, Yes/No). Для детектирования текста используется предобученная модель CRAFT (на основе FCN), которая локализирует отдельные области символов, связывает обнаруженные символы с экземпляром текста и находит баундбоксы текста. Полученные баундбоксы расширяются на 1.5% для улучшения качества распознавания текста и сохраняются в виде отдельных изображения. Затем изображения обрабатываются средствами библиотеки OpenCV. Распознавание текста на изображении и его перевод в строковые символы осуществляется с помощью пакета PyTesseract  
2. Фигуры. Перед распознаванием фигур происходит обработка изображения средствами OpenCV: стирание всего текста и морфологическая обработка - дилатация (растягивание, операция расширения) и эрозия (размывание, операция сужения). Цель этих операций - удаление шумов, изначально содержащихся в изображении или возникших в результате операции стирания текста. Детектирование фигур происходит путем применения функции findContours из OpenCV, возвращающей сгруппированные наборы точек, которые являются точками контура,  а баундбоксы - функцией minAreaRect, пытающейся найти прямоугольник максимального размера, который может вписаться в заданный замкнутый контур. Для улучшения качества распознавания баундбоксы расширяются нам 3%, затем к вырезанным изображениями добавляются белые поля шириной 20 пикселей. Распознавание фигур происходит так: на вырезанном изображении ищутся контуры разумной для геометрической фигуры площади, последний из таких контуров является минимальным по площади и самым "правильным" - по нему вычисляется аппроксимационный контур функцией approxPolyDP, количество сторон которого определяет форму фигуры (четырехугольник, шестиугольник, шестиугольник со стороной наибольшей длины снизу или сверху, овал)
3. Стрелки. Обнаруженные фигуры закрашиваются белыми прямоугольниками внутри расширенных баундбоксов, и итоговое изображение содержит только отрезки стрелок. Детектирование линий (отрезков) производится с помощью алгоритма Canny, а распознавание - нахождение координат точек начала и конца - с помощью метода Хафа, функцией HoughLinesP 
4. Точки вхождения стрелок. Имеются в виду точки вхождения горизонтальных отрезков стрелок в вертикальные - они находятся путем перебора отрезков и проверки, лежит ли конец горизонтального отрезка в окрестности координат вертикального отрезка. Поскольку детектирование отрезков работает с погрешностью и на месте одного отрезка может быть несколько отрезков, точек вхождения стрелок может быть больше, чем есть на блок-схеме по смыслу. Производится иерархическая кластеризация точек методом hcluster из библиотеки scipy, затем для для каждого кластера считается его центр масс - эти центры и есть искомые точки вхождения, они является окончаниями циклов. Окончанием цикла также является шестиугольник, с наиболее длинной стороной сверху и овал в конце блок-схемы.
5. Структура данных. Список словарей элементов схемы. Ключи и значения: текст (text) - название фигуры (может быть и точка), бокс (box) - набор координат углов баундбокса фигуры, координаты (coord) - координаты центра тяжести баунбокса, код (code) - текст внутри фигуры, также флаг is_visited (отметка о посещении узла), по умолчанию False

### Форматы хранения элементов схемы
1. Текст. Координаты баундбоксов: x0 y0 x1 y1 x2 y3 x3 y3 - порядок обхода от верхнего левого угла по часовой стрелке (файл output_text_box.txt). Распознанный текст построчно (файл output_text.txt). Промежуточные данные: auto_text - области с автоматически найденными баундбоксы, corrected_text - области со слегка расширенными баундбоксами
2. Фигуры. Координаты баундбоксов: x0 y0 x1 y1 x2 y3 x3 y3 - порядок обхода от верхнего левого угла по часовой стрелке (файл output_figure_box.txt). Названия фигур построчно (output_figure.txt). Промежуточные данные: corrected_figures - области со слегка расширенными баундбоксами 
3. Стрелки. Координаты начала и конца отрезков, составляющих стрелки: x0 y0 x1 y1 (файл output_lines_box.txt), итоговое изображение edges.png
> Все директории с промежуточными результатами и полезными данными (координаты баундбоксов/линий) создаются автоматически
___
### Генерация кода по элементам схемы
1. Каждый шаг схемы представляется в виде некоторого класса, который наследуется от класса генератора кода
2. Генератор кода представляет собой дерево, которое хранит указатель на начало схемы, а также имеет метод генерации кода
3. Каждый класс, представляющий определенный шаг схемы также имеет метод генерации кода по данным, которые у него есть
4. Общими атрибутами всех классов шагов является указатель на следующий и предыдущий шаг, а также некоторые текстовые данные, нужные для генерации кода
5. По сути генерация кода классом генератора представляет из себя последовательный проход по экземплярам классов шагов и вызова у них встроенного метода генерации кода
6. Отметим, что некоторые шаги (начало условия или начало цикла) порождают новые деревья, поэтому код генерируется рекурсивно
7. При построении кода осуществляется обход схемы сверху вниз и сева направо, в зависимости от того, имеет ли данный элемент соседа снизу (навравление yes) или соседа справа (направление no). Сначала производится обход элементов в направлении крайнего справа цикла и рассматривается его ветка no до точки окончания цикла, флаг is_visited обращается в True, попутно все циклы (их начальные элементы) кладутся на стек, при завершении обхода в направлениях no и yes циклы со стека удаляются
8. В зависимости от элемента (фигуры) и текста внутри него генерируется соответсвующая строка кода на языке Python