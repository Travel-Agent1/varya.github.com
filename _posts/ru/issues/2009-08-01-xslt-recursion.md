---

title: Эффективное использование рекурсии в XSL

categories: ru issues

layout: post

---

Перевод статьи <a href="http://www.ibm.com/developerworks/xml/library/x-xslrecur/">Use recursion effectively in XSL</a>.
Введение в XSL-рекурсию и приёмы для оптимизации её использования.

Эффективное и рациональное использование XSL-преобразований требует понимания, как использовать XSL в качестве функционального языка, что означает понимание рекурсии. Эта статья знакомит с ключевыми идеями рекурсии и особенностей её использования в XSL. Также объяснены приёмы для оптимизации преобразования XML и избегания ошибок при использовании рекурсии. Каждая идея или метод сопровождаются примерами кода.<!--more-->

Сейчас в мире преобладают императивные языки программирования. Наиболее популярные языки - Java, C (и его различные виды), Visual Basic и другие - на высоком абстрактном уровне работают, в основном, одинаково: вы задаёте некоторые переменные, вызываете функции или операторы, которые меняют эти переменные, и возвращаете результат. Это сильное приближение к программированию, и это, конечно, не всё.

Языки программирования другой породы, пока менее привычной, по крайней мере также сильны как их процедурные "коллеги". Эти языки названы <em>функциональными</em> или <em>декларативными</em> языками. Программы, написанные на этих языках, могут только однажды определить переменные, и никогда не могут поменять хранимое значение однажды определённой переменной. Язык программирования XSL и функциональный, и декларативный. Это означает, что разработчики, привыкшие писать на Java или C, и изучающие XSL часто чувствуют себя не в своей тарелке, применяя самые передовые особенности XSL.

Из-за роста важности как приложений, так и web-разработчиков и завязанности на XSL-технологию, эффективное использование XSL-преобразований не может быть проигнорировано. Так, наиболее важно научиться, как программировать в декларативной манере. Это подразумевает близкое знакомство с <strong>рекурсией</strong>, основанное на методах её эффективного использования и рационального решения текущих задач.

<h2><strong>Введение в рекурсию на примере</strong></h2>

Через использование рекурсивных функций декларативные языки способны предоставлять функциональность, подобную функциональности их императивных "коллег". Это не значит, что императивные языки не могут использовать рекурсию. Большинство - могут. Разница в том, что декларативные языки используют рекурсию как основное средство деятельности, тогда как чаще всего это просто свойство в императивных языках. 

Рассмотрим функцию, которая высчитывает факториал от некоторого положительного целого. Вкратце, факториал числа - это число, полученное путём умножения всех предшествующих ему чисел. Так, факториал от 4 (или 4!) - 1*2*3*4 = 24. Распечатка 1 показывает один из способов написать эту функцию на Java:

<em>Распечатка 1. Решение задачи про факториал с использованием цикла на Java</em>
[cc lang="java"]
public int factorial(int number) {
    if (number <= 1) return 1;
    int result = 1;
    for (int i=1; i <= number; i++) {
        result *= i;
    }
    return result;
}
[/cc]

Пока это совершенно разумный код, он действительно пользуется тем фактом, что вы можете переопределить значения переменных в Java-программе. Без этой роскоши вы можете только решать задачу определением функций, как в Распечатке 2.

<em>Распечатка 2. Решение задачи про факториал с использованием рекурсии на Java</em>
[cc lang="java"]
public int factorial(int number) {
    if (number <= 1) return 1;
    return number * factorial(number-1);
}
[/cc]

Результаты вызова этих двух реализаций вычисления факториала одинаковы. Вызов функции <em>factorial()</em> в теле метода показывает, что вторая реализации использует рекурсию. Этот метод может быть визуально представлен как лестница. Программа многократно спускается вниз, вызывая саму себя, собирая статическую информацию для каждого вызова пока не упрется в условие выхода из рекурсии. Достигнув конца, она идёт обратно, возвращая различные состояния уже сделанных вызовов, объединяя информацию, собранную на пути вниз, для получения финального результата. Пример про факториал показан на рисунке.
<img src="http://www.ibm.com/developerworks/xml/library/x-xslrecur/steps.gif" alt="Представление рекурсии в виде лестницы" />

В общем случае, рецепт создания рекурсивной функции включает в себя три ингредиента: условие выхода, код операции и рекурсивный вызов самого себя. В случае факториала, я вычисляю его для n путём вычисления факториала для n-1, n-2, n-3 и т.д. пока функция не достигнет условия выхода из серии - значения 1.

Так, если сейчас идея рекурсии ещё кажется немного смущающей, вы могли бы прервать чтение этой статьи и либо проследовать по некоторым ссылкам на эту тему в разделе <a href="http://www.ibm.com/developerworks/xml/library/x-xslrecur/#resources">Ресурсы</a>, либо попробовать написать несколько функций самостоятельно (например, напишите функцию для расчёта n-го числа из ряда Фибоначчи).

<h2><strong>Общие использования рекурсии в XSL</strong></h2>

Два общих сценария использования рекурсии в XSL такие:
<ul>
  <li>Когда у вас есть набор повторяющихся значений в исходном XML, и вы хотите, чтобы результат преобразования отражал что-то, касающееся всех этих значений. Например, если у вас есть каталог товаров в XML, и вы хотите представить эти товары в соответствии с общей ценой, вы захотите найти совокупную цену, используя рекурсивный шаблон.</li>
  <li>Когда исходный XML содержит число <em>x</em> в теге, например &lt;countTo number="5"/&gt;, и вы хотите представить некоторую информацию <em>x</em> раз в результирующей выдаче. Факториал - типичный пример такого случая, но я рассмотрю более сложный пример немного позже.</li>
</ul>
Я покажу рекурсивное решение, написанное на XSL для обоих сценариев, но сначала давайте взглянем на пример про факториал, решенный на XSL:

<em>Распечатка 3. Решение задачи про факториал с использованием рекурсии на XSL</em>
[cc lang="xml"]
<xsl:template name="factorial">
  <xsl:param name="number" select="1"/>
  <xsl:choose>
    <xsl:when test="$number <= 1">
      <xsl:value-of select="1"/>
    </xsl:when>
    <xsl:otherwise>
      <xsl:variable name="recursive_result">
        <xsl:call-template name="factorial">
          <xsl:with-param name="number" select="$number - 1"/>
        </xsl:call-template>
      </xsl:variable>
      <xsl:value-of select="$number * $recursive_result"/>
    </xsl:otherwise>
  </xsl:choose>
</xsl:template>
[/cc]

Этот шаблон содержит все "игредиенты" из рецепта для Java-рекурсии. Здесь есть условие выхода (проверка на равенство единице или нет): если условие выхода не выполняется, шаблон делает рекурсивный вызов самого себя (хранимый в переменной <em>recursive_result</em>). Наконец, для получения конечного результата совершается операция умножения между исходным числом и рекурсивно полученным значением. Вы также могли заметить, что XSL частно требует намного больше кода, чем решение той же задачи в других языках. Это всего лишь часть неоднозначностей в языке преобразования, который согласовывается с XML.

<h3><strong>Рекурсия, вариант 1: Суммирование сквозь множество элементов</strong></h3>

Теперь вы можете применить такой же метод, как и в примере с факториалом из <em>Распечатки 3</em>, чтобы найти общую стоимость товаров в XML-каталоге. Фактически, решение аналогично решению задачи с факториалом, за исключением того, что вы будете использовать сложение вместо умножения и набор узлов как исходную информацию вместо числа.

Предположим, у вас есть каталог товаров, который выглядит походим на XML в <em>Распечатке 4</em> (кстати, вы можете скачать весь XML и XSL, и весь другой код из этой статьи по ссылке в <a href="http://www.ibm.com/developerworks/xml/library/x-xslrecur/#resources">Ресурсах</a>).

<em>Распечатка 4. Пример XML с продуктами</em>
[cc lang="xml"]
<Products>
  <Product>
    <Name>Gadget</Name>
    <Price>$10.00</Price>
  </Product>
  <Product>
    <Name>Gizmo</Name>
    <Price>$7.50</Price>
  </Product>
  ...
 </Products>
[/cc]

Цель в том, чтобы добавить первую цену из этого списка товаров к сумме всех других цен, пока вы не получите суммарный результат. Вы можете осуществить это при помощи XSL, показанного в <em>Распечатке 5</em>.

<em>Распечатка 5. Проход по узлам с использованием обычной XSL-рекурсии</em>
[cc lang="xml"]
<xsl:template name="priceSum">
  <xsl:param name="productList"/>
    <xsl:choose>
      <xsl:when test="$productList">
        <xsl:variable name="recursive_result">
          <xsl:call-template name="priceSum">
            <xsl:with-param name="productList"
              select="$productList[position() > 1]"/>
          </xsl:call-template>
        </xsl:variable>
        <xsl:value-of
          select="number(substring-after($productList[1]/Price,'$'))
                  + $recursive_result"/>
      </xsl:when>
      <xsl:otherwise><xsl:value-of select="0"/></xsl:otherwise>
  </xsl:choose>
</xsl:template>
[/cc]
В этом случае вы вызываете функцию на постепенно уменьшающемся списке товаров, каждый раз добавляя первый элемент в список к рекурсивному вызову на остальных элементах списка. Когда список получается пустым, условие выхода достигнуто и вы добавляете ноль к результату, что не изменит сумму. Этот метод работы на всё меньшем и меньшем наборе узлов может быть использован во многих случаях.

<h3><strong>Рекурсия, вариант 2: Итерация на числе</strong></h3>

Вторая задача, с которой сталкиваются XSL-программисты, возникает, когда число содержится в некоторых случаях в исходном XML, и программисту нужно решить задачу это число раз. Например, XML может выключать информацию о сетке с некоторыми числами строк и столбцов, и преобразование необходимое для обеспечения визуального расположения сетки в HTML или каком-либо другом формате.

Решение, как правило используемое разработчиками, незнакомыми с рекурсивными методами - это написать программу, которая разбирает XML и дополняет элементами для каждого ряда или столбца. Однажды написанный элемент &lt;xsl:for-each&gt; может решить задачу без рекурсивных шаблонов. Несмотря на то, что это решение обеспечивает подходящую выдачу, у него есть большой недостаток. Этот метод на самом деле удваивает объём работы для обеспечения финального результата преобразования. Этот недостаток чрезвычайно заметен в клиент-серверной модели, когда сервер посылает XML и XSL для преобразования на клиентской стороне. В таком случае всё продуктивности, обычно реализованные на сервере, теряются за счёт увеличения XML.

Лучшее решение - использовать рекурсивные шаблоны, чтобы выполнять работы полностью в рамках преобразования. Рассмотрим получение таблицы умножения из XML-элемента в виде HTML. Входная информация будет содержать следующую строку:
[cc lang="xml"]
<MultiplicationTable Rows="3" Columns="4"/>
[/cc]
Цель состоит в том, чтобы преобразовать это в HTML, представляющий таблицу, похожую на:
<table border="1" cellpadding="5" cellspacing="0">
<tr>
<td></td>
<td><strong>1</strong></td>
<td><strong>2</strong></td>
<td><strong>3</strong></td>
<td><strong>4</strong></td>
</tr>
<tr>
<td><strong>1</strong></td>
<td>1</td>
<td>2</td>
<td>3</td>
<td>4</td>
</tr>
<tr>
<td><strong>2</strong></td>
<td>2</td>
<td>4</td>
<td>6</td>
<td>8</td>
</tr>
<tr>
<td><strong>3</strong></td>
<td>3</td>
<td>6</td>
<td>9</td>
<td>12</td>
</tr>
</table>

Этот пример кое в чём более сложный, чем первые два. В этом случае вы не просто возвращаете функциональный результат, представляющий решение проблемы, но вы всего лишь выдадите HTML как шаг рекурсивного процесса. Также вам необходимо соединить вместе два рекурсивных шаблона - один для рядов, другой для столбцов.

<em>Распечатка 6. Повторение по числу с использованием основной XSL-рекурсии</em>
[cc lang="xml"]
<xsl:template name="drawRow">
  <xsl:param name="currentRow"/>
  <xsl:param name="totalRows"/>
  <xsl:param name="totalCols"/>
  <xsl:if test="$currentRow <= $totalRows">
    <tr>
      <xsl:call-template name="drawCell">
        <xsl:with-param name="currentRow" select="$currentRow"/>
        <xsl:with-param name="currentCol" select="0"/>
        <xsl:with-param name="totalCols" select="$totalCols"/>
      </xsl:call-template>
    </tr>
    <xsl:call-template name="drawRow">
      <xsl:with-param name="currentRow" select="$currentRow  + 1"/>
      <xsl:with-param name="totalRows" select="$totalRows"/>
      <xsl:with-param name="totalCols" select="$totalCols"/>
    </xsl:call-template>
  </xsl:if>
</xsl:template>

<xsl:template name="drawCell">
  <xsl:param name="currentRow"/>
  <xsl:param name="currentCol"/>
  <xsl:param name="totalCols"/>
  <xsl:if test="$currentCol <= $totalCols">
    <xsl:variable name="bgColor">...</xsl:variable>
    <xsl:variable name="value">...</xsl:variable>
    <td bgcolor="{$bgColor}" align="center" valign="top">
      <xsl:value-of select="$value"/>
    </td>
    <xsl:call-template name="drawCell">
      <xsl:with-param name="currentRow" select="$currentRow"/>
      <xsl:with-param name="currentCol" select="$currentCol + 1"/>
      <xsl:with-param name="totalCols" select="$totalCols"/>
    </xsl:call-template>
  </xsl:if>
</xsl:template>
[/cc]

В этом случае я использую два отдельный рекурсивных шаблона. Вначале рекурсивно рисую ряды пока не достигну условия выхода. Этот шаблон использует второй рекурсивный шаблон, чтобы нарисовать ячейки для каждого столбца в ряду. Этот общий метод применим для многоуровневого представления данных.

<h2><strong>Оптимизация паттернов рекурсии в XSL</strong></h2>

К сожалению, использование рекурсии во многих XSL-процессорах имеет один значительный недостаток при работе с большими объемами данных. Также слишком легко достигнуть переполнения стека (Stack Overflow) или нехватки памяти (Out of Memory) с функциями, которые достигают глубины порядка 10000. Если вы пытаетесь суммировать цены в каталоге с более чем 10000 товарами, вы можете столкнуться с проблемами.

К счастью, есть пути по оптимизации рекурсивных функций, чтобы уменьшить или даже исключить глубину рекурсии, вытекающую из заданного шаблона. Оставшаяся часть этой статьи представляет четыре способа оптимизировать то, как вы пишете ваши рекурсивные решения на XSL.

<h3><strong>Оптимизация рекурсии 1. Разделяй и властвуй</strong></h3>

Вспомните первый пример, в котором мы пытались найти общую цену, суммировав все цены для нескольких товаров в XML-каталоге. Первоначальный подход - это добавить цену первого товара к сумме всех остальных, и делать это рекурсивно, пока вы не переберёте весь лист. При использовании этого подхода глубина рекурсии равна числу товаров в списке. Сейчас цель состоит в том, чтобы изменить метод так, чтобы вы получили тот же результат без вхождения в рекурсию так глубоко.

Другой подход к этой проблеме - это разделить список на множество меньших частей и делать так рекурсивно, пока вы не разделите каждую часть на отдельные продукты. Самый простой путь представить это - разделение списка напополам, делая рекурсивный вызов на каждой половине и тогда сделать отдельные рекурсивные вызовы на двух получившихся списках. Это позволит завершить всю работу на одной половине,  до того как какая-то работа будет сделана на другой половине. Процесс обработки будет древовидный, а не линейный (похожий на лестницу) из первого решения. <em>Рисунок 2</em> показывает это рекурсивное дерево.
<img src="http://www.ibm.com/developerworks/xml/library/x-xslrecur/tree.gif" alt="" />

Используя подход "разделяй и властвуй", вы не ощутите никакой экономии, в цифрах, по числу операций сложения, которые должны быть выполнены. Экономия будет заключаться в том, что операции сложения не ожидают того, чтобы все числа были получены и поставлены в очередь в стек рекурсии для выполнения операции. XSL-код в <em>Распечатке 7</em> показывает такой способ.

<em>Распечатка 7. Прохождение по узлам с использованием метода "разделяй и властвуй"</em>
[cc lang="xml"]
<xsl:template name="priceSumDivideAndConquer">
  <xsl:param name="productList"/>
    <xsl:choose>
      <xsl:when test="count($productList) = 1">
        <xsl:value-of
          select="number(substring-after($productList[1]/Price,'$'))"/>
      </xsl:when>
      <xsl:when test="$productList">
        <xsl:variable name="halfIndex"
          select="floor(count($productList) div 2)"/>
        <xsl:variable name="recursive_result1">
          <xsl:call-template name="priceSumDivideAndConquer">
            <xsl:with-param name="productList"
              select="$productList[position() &lt;= $halfIndex]"/>
          </xsl:call-template>
        </xsl:variable>
        <xsl:variable name="recursive_result2">
          <xsl:call-template name="priceSumDivideAndConquer">
            <xsl:with-param name="productList"
              select="$productList[position() &gt; $halfIndex]"/>
          </xsl:call-template>
        </xsl:variable>
      <xsl:value-of select="$recursive_result1 + $recursive_result2"/>
    </xsl:when>
    <xsl:otherwise><xsl:value-of select="0"/></xsl:otherwise>
  </xsl:choose>
</xsl:template>
[/cc]
Операция сложения происходит каждый раз, когда разделённый список достигает длины 1 (это является условием выхода), и производится до того, как какая-нибудь обработка произойдёт со второй частью листа. При использовании этой техники глубина рекурсии никогда не превысит log<sub>2</sub> от количества элементов в списке, что, конечно, экспоненциально увеличивает возможное число товаров в списке для этого подсчёта.

<h3><strong>Оптимизация рекурсии 2. Сегментация</strong></h3>

Несмотря на невероятную экономию глубины рекурсии, обеспеченные методом "Разделяй и властвуй", у него действительно есть недостатки. Для списков длиной не выражается числом степени двойки, этот способ излишне накладный на концах рекурсионного дерева.

Другой подход, который похоже уменьшает глубину рекурсии, работающий на итерациях через сегменты узлов предопределённой длины, предпочтительнее, чем деление узлов надвое каждый раз. Этот подход построен на базовой рекурсивной методике, но представлен внешним шаблоном, который действует как менеджер сегментации.

Роль менеджера сегментации состоит в том, чтобы разбить большую задачу на маленькие, которые могут быть проработаны без появления утечек памяти. Менеджер хранит результаты этих маленьких заданий и в тот момент, когда они все завершены, выполняет необходимую операцию над этими результатами.

Вы можете видеть пример техники Сегментации в шаблоне <em>Распечатки 8</em>, который берёт список узлов и длину сегмента и использует оригинальный шаблон для цен на этих маленьких частях.

<em>Распечатка 8. Обход узлов с использованием техники сегментации</em>
[cc lang="xml"]
<xsl:template name="priceSumSegmented">
  <xsl:param name="productList"/>
  <xsl:param name="segmentLength" select="5"/>
  <xsl:choose>
    <xsl:when test="count($productList) > 0">
      <xsl:variable name="recursive_result1">
        <xsl:call-template name="priceSum">
          <xsl:with-param name="productList"
            select="$productList[position() <= $segmentLength]"/>
        </xsl:call-template>
      </xsl:variable>
      <xsl:variable name="recursive_result2">
        <xsl:call-template name="priceSumSegmented">
          <xsl:with-param name="productList"
            select="$productList[position() > $segmentLength]"/>
        </xsl:call-template>
      </xsl:variable>
      <xsl:value-of select="$recursive_result1 + $recursive_result2"/>
    </xsl:when>
    <xsl:otherwise><xsl:value-of select="0"/></xsl:otherwise>
  </xsl:choose> 
</xsl:template>
[/cc]
Использование подхода Сегментации лучше, чем подхода "Разделяй и властвуй" уменьшает накладные расходы в большинстве случаев, но не предлагает особенной выгоды по глубине рекурсии. Какой же следует использовать? Ответ - как вы можете увидеть по следующей оптимизации - оба.

<h3><strong>Оптимизация рекурсии 3. Сочетание Сегментации и "Разделяй и властвуй"</strong></h3>

Вы можете объединить техники "Разделяй и властвуй" и Сегоментации, чтобы
<ol>
  <li>увеличить экномию за счёт снижения глубины рекурсии (включая накладную обработку, связанную 
с каждый добавлением уровня используемой рекурсии)</li>
  <li>избежать накладных расходов на уровне листочков рекурсионного дерева</li>
</ol>

В этом способе пороговый уровень передаётся в рекурсивный шаблон, через список узлов (или другие данные, которыми опрерируют). Здесь "Разделяй и властвуй" изменён, чтобы работать как менеджер сегментации. Если размер переданного списка больше, чем пороговое значение, список делится надвое и шаблон рекурсивно вызывается для обеих половин. Другими словами, вы используете основной рекурсивный шаблон, чтобы вычислять результаты, как и в Сегментации.
<em>Распечатка 9</em> показывает этот способ на следующих XSL-шаблонах.

<em>Распечатка 9. Обход узлов с использованием составной техники "Разделяй и властвуй" и Сегментации</em>
[cc lang="xml"]
<xsl:template name="priceSumCombination">
  <xsl:param name="productList"/>
  <xsl:param name="threshold" select="5"/>
  <xsl:choose>
    <xsl:when test="count($productList) > $threshold">
      <xsl:variable name="halfIndex"
        select="floor(count($productList) div 2)"/>
      <xsl:variable name="recursive_result1">
        <xsl:call-template name="priceSumCombination">
          <xsl:with-param name="productList"
            select="$productList[position() <= $halfIndex]"/>
        </xsl:call-template>
      </xsl:variable>
      <xsl:variable name="recursive_result2">
        <xsl:call-template name="priceSumCombination">
          <xsl:with-param name="productList"
            select="$productList[position() > $halfIndex]"/>
        </xsl:call-template>
      </xsl:variable>
      <xsl:value-of select="$recursive_result1 + $recursive_result2"/>
    </xsl:when>
    <xsl:otherwise>
      <xsl:call-template name="priceSum">
        <xsl:with-param name="productList" select="$productList"/>
      </xsl:call-template>
    </xsl:otherwise>
  </xsl:choose>
</xsl:template>
[/cc]

<h3><strong>Оптимизация рекурсии 4. Tail Recursion</strong></h3>

У вас уже есть какой-то опыт по отношению к рекурсии и вы, возможно, удивлены, почему метод Tail Recursion не первый в способах оптимизации, с тех пор как этот способ может устранить глубокую рекурсию полностью без каких-либо накладных расходов. Несмотря на преимущества, которые этот способ предлагает, он требует, чтобы XSLT-процессор, производя эту трансформацию, распознавал присутствие этой техники в XSL-коде и изменял бы своё поведение для обеспечения этой техники. К сожалению, большинство XSL-процессоров не предлагают такую возможность.

Хорошая новость состоит в том, что непомерное одобрение XSL в бизнесе и научном мире означает, что это ограничение просуществует недолго. По этой причине для разработчиков важно понимать, как происходит отрубание хвоста, как оно может исключить проблемы с памятью без задерживания представления в любом достойном виде.

Основная идея Tail Recursion - это избавление от хранения какой-либо статической информации в рекурсивных шагах. Вся информация, которая нужна, на каждом шаге передаётся как параметры функции вместо того, чтобы храниться на более высоком уровне в стеке рекурсии. Это позволяет XSLT-процессору работать с рекурсивной функцией как с циклом в процедурном языке.

Взгляните на <em>Распечатку 1</em>0 примера с ценами с Tail Recursion.

Распечатка 10. Проход по узлам с использованием Tail Recursion
[cc lang="xml"]
<xsl:template name="priceSumTail">
  <xsl:param name="productList"/>
  <xsl:param name="result" select="0"/>
  <xsl:choose>
    <xsl:when test="$productList">
      <xsl:call-template name="priceSumTail">
        <xsl:with-param name="productList"
          select="$productList[position() > 1]"/>
        <xsl:with-param name="result"
          select="$result + 
                  number(substring-after($productList[1]/Price,'$'))"/>
      </xsl:call-template>
    </xsl:when>
    <xsl:otherwise><xsl:value-of select="$result"/></xsl:otherwise>
  </xsl:choose>
</xsl:template>
[/cc]
Прибавление дополнительного значения переменной создаёт всю разницу. Вместо накомления чисел для сложения на разных рекурсивных шагах, сложение производится на каждом этапе и результат передаётся дальше как параметр для следующего шага рекурсии. Токовый XSLT-процессор просто перезаписывает участок памяти, содержащий значение переменной, при каждом вызове так же, как эта переменная перезаписывалась бы в случае Java или C кода. Этот способ позволяет языку пользоваться преимуществами и декларативных, и императивных языков без изменения сути XSLT как языка программирования.

<h2><strong>Заключение</strong></h2>
Как только вы поняли рекурсию, декларативный стиль программирования на XSLT будет не препятствием, а эффективным путем расширения возможностей преобразования XML. Остаётся единственный вопрос - какой тип рекурсии лучший для каждой отдельно взятой ситуации.

Вообще, задачи, работающие для малых объёмов данных, не требуют применения какой-либо из этих оптимизационных техник. Хотя, если малость объёма данных не гарантирована, выбор ограничен технологией, используемой в преобразовании. Если ваш XSLT-процессор распознаёт Tail Recursion, то лучше всего использовать этот способ. Если вы не можете убедиться, что преобразовательня технология умеет распознавать Tail Recursion, то наиболее предпочтительна комбинированная техника. Пороговое значение обычно лежит в диапазоне от 5 до 30 в зависимости от задачи.

Поначалу рекурсия может быть трудной для понимания идеей, но её полезность и элегантность становятся яснее по мере использования.
