Lab4controls
============

Лабораторная 4 - пользовательские компоненты в Windows Forms


Упражнение
=======

1. Создайте проект Windows Forms.
2. Добавьте к проекту пользовательский компонент - "Add" - "New Item" - "Custom Control". Назовите компонент ``RedButton``.
3. Посмотрите сгерерированный код и окно дизайнера компонента (2 .cs файла, ViewCode)
4. Поменяйтe базовый класс RedButton с Control на Button (в RedButton.cs).
5. Измените в свойствах  RedButton (дизайнером) цвет фона (BackColor) на красный (Red), увеличьте шрифт, измените размер по умолчанию 
на 75;75
6. Подключим только что созданный компонент к ToolBox (если его там ещё нет): 
   - Скомипилируйте и запустите проект
   - Toolbox - Choose Items (в контекстном меню) - .NET Components - Browse - Выбрать exe-файл (в нём сборка .NET) проекта. Активировать компонент RedButton.
7. Поместите на главную форму несколько элементов RedButton, убедитесь, что они отображаются как нужно.   
8. Добавьте к кнопкам возможность зачеркивания:

    ```c#
    public bool Crossed{ get;set; } // добавить СВОЙСТВО
    ...
    //в OnPaint добавляем рисование зачеркивающих линий
    if (Crossed)
    {
      Graphics g = pe.Graphics;
      Pen p = new Pen(Color.Yellow, 3);
      g.DrawLine(p, 0, 0, this.Width, this.Height);
      g.DrawLine(p, 0, this.Height, this.Width, 0);
    }
    ```
    
9. Попробуйте менять свойство Crossed у кнопок в дизайнере главной формы (после перекомпиляции проекта).
10. Исправьте недостаток программы - замедленную реацию на изменение свойства. 
    - Если Crossed изменяется при присваивании, нужно вызвать ``Invalidate`` (потребовать перерисовки чуть позже).

    ```c#
    bool c;
    public bool Crossed
    {
      get { return c; }
      set { ... value ... Invalidate(); ... }
    }
    ```
11. В результате, зачёркивание в дизайнере форм должно появляться одновременно с изменением свойства ``Crossed``.

Задание
=======

1. Добавьте пользовательский компонент ``Grid`` (оставить наследником класса ``Control``)
2. Сначала компонент должен рисовать сетку из линий заданного размера
  - введите свойства ``RowCount``, ``ColumnCount``, ``CellSize``

     ```c#
     int rows=5, cols=3, cellSize=30;
     public int RowCount { get { return rows; } set { (проверки) rows=value (Invalidate(), если нужно) }
     ```

  - в функции ``OnPaint`` добавить рисование сетки (шаг равен ``CellSize`` точек, количество горизонтальных линий определяется ``ColumnCount``, вертикальных - ``RowCount``. Используется функция ``DrawLine``, объект ``Pen`` можно сделать полем класса ``Grid``.
 
3. Добавьте чуть более сложное свойство - цвет линии.
    
    ```c#
    Pen p = new Pen(  .... )
    public Color LineColor { использовать p.Color }
    ```

4. Следующая цель - печать чисел в ячейках.
    - добавьте поле ``data`` - двумерный массив чисел ``int[,] data =...``
    - пока можно выделить память про запас (1000x1000) (иначе придётся изменять размер массива вместе c RowCount/ColumnCount)
    - добавьте функцию установки элемента массива:
    
       ```c#
        public void SetCell(int i, int j, int value)
       ```
    
    - добавьте **индексатор**, который делает то же самое:
    
       ```c#
          public int this[int i,int j] {
            get { ...  }  set { ... value ... }
          }
       ```
    
    - обеспечьте вывод чисел из массива в центры ячеек (используйте ``DrawString`` и цикл в функции On_Paint)

       ```c#
          g.DrawString(Convert.ToString(data[i,j]), 
                  Font, // шрифт, выбранный в дизайнере
                  Brushes.Black, // цвет цифр
                  x, y // координаты надписи
                  );
       ```

5. Добавьте обработчик события на какую-нибудь кнопку, который занесёт случайное число в случайную ячейку.
(поле ``Random r=new Random();``, используем ``r.next(N)`` для числа от 0 до N-1)

6. Добавьте событие реакции на нажатие мыши. Должно порождаться событие, аргументом которого будет положение ячейки.
     - добавьте тип события (делегат) к классу Grid:
     
     ```c#
     public delegate void CellHandler(Object sender, int row, int column);
     ```
     
     - объявите событие:
     
     ```c#
     public event CellHandler CellClick;
     ```
     
     (событие появится в таблице дизайнера)
     
     - переопределите функцию обработки нажатия мыши у класса Grid (вызовите в ней событие с правильными номерами строки и столбца)
     
     ```c#
     protected override void OnMouseClick(MouseEventArgs e)
     {
        base.OnMouseClick(e);
        //  проверьте, что кликнули внутрь таблицы, а не мимо
        if (CellClick!=null) CellClick(this, e.Y / cellSize, e.X / cellSize);
     }
     ```
     
6. Добавьте обработчик события, заменяющий число в нажатой ячейке на случайное.

7. Добавьте в класс Grid режимы работы (показ чисел, раскраска или числа с раскраской)
    - добавьте перечислимый тип (enum) с тремя значениями:
     
     ```c#
     public enum GridDisplayMode { Numbers, Colors, NumberAndColors };
     ```
    
    - добавьте cвойство этого типа:
    
     ```c#
     public GridDisplayMode DisplayMode {get; set;}
     ```
     
    - добавьте свойство-массив цветов (для упрощения - 5 шт.):
    
     ```c#
     public Color[] Colors {get; set; } 
     ```
    
    - инициализируйте ``Colors`` в конструкторе (5 произвольных цветов)
    - убедитесь, что дизайнер позволяет редактировать цвета и выбирать ``DisplayMode``
    - в функции On_Paint в зависимости от DisplayMode рисуйте закрашенный прямоугольник (FillRectangle, цвет ``Colors[data[i,j]]``), текст, или вместе
    - функции FillRectangle нужен объект Brush, его можно создать из цвета ``new SolidBrush(c)`` (можно ввести вспомогательный массив)
    - если элемент массива вне диапазона доступных номеров цветов, то ячейка не закрашивается.

9. Добавьте на форму переключатель режимов - ComboBox. 
     - Его элементами (Items) в конструкторе Form1 назначьте ``Grid.GridDisplayMode.Colors`` и остальные 2 режима (это объекты).
        ```c#
        comboBox1.Items.Add(Grid.GridDisplayMode.Colors);
        ...
        ```

     - обработчиком события ``SelectedIndexChanged`` для ``сomboBox1`` сделайте функцию с операторами
         
         ```c#
         grid1.DisplayMode = (Grid.GridDisplayMode) comboBox1.SelectedItem;
         grid1.Invalidate();
         // что если comboBox1.SelectedItem == null ?
         ```
10. Проверьте, что всё работает :)
