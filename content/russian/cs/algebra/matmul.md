---
title: Задачи на умножение матриц
weight: -8
authors:
- Сергей Слотин
- Максим Иванов
prerequisites:
- binpow
- matrix
date: 2021-08-30
---

Когда мы говорили о линейных функциях и матрицах, все примеры были про геометрию. Так просто проще думать: представлять в уме повороты и растяжения векторов на плоскости легче, чем рассуждать о свойствах абстрактных *n*-мерных пространств. Однако мы программисты, и нас в основном интересуют задачи вне геометрии.

Большинство применений в контексте олимпиад опираются на ассоциативность матричного умножения:

$$
ABC = (AB)C = A(BC)
$$

Если задачу можно свести к возведению матрицы $A$ в какую-то большую степень $n$ — то есть к домножению единичной матрицы $n$ раз на матрицу $A$ — то можно просто воспользоваться свойством ассоциативности и посчитать результат бинарным возведением в степень:

$$
A^8 = A^{2^{2^{2}}} = ((AA)(AA))((AA)(AA))
$$

Такой подход имеет множество применений в динамическом программировании, комбинаторике, теории вероятностей и математике в целом.

## Линейные рекурренты

**Задача.** Дано число $n < 10^{18}$. Требуется вычислить $n$-ное число Фибоначчи:

$$
\begin{aligned}
    f_0   &= 0
\\  f_1   &= 1
\\  f_{n} &= f_{n-1} + f_{n-2}
\end{aligned}
$$

Заметим, что вычисление очередного числа Фибоначчи выражается как линейная функция от двух предыдущих чисел Фибоначчи — а именно, как их сумма.

Это означает, что мы можем построить матрицу $2 \times 2$, которая будет соответствовать этому преобразованию: как по двум соседним числам Фибоначчи $(f_n, f_{n+1})$ вычислить следующее число, то есть перейти к паре $(f_{n+1}, f_{n+2})$.

$$
\begin{pmatrix}
f_{n+1} \\
f_{n+2} \\
\end{pmatrix}
=
\begin{pmatrix}
0 + f_{n+1} \\
f_{n} + f_{n+1} \\
\end{pmatrix}
=
\begin{pmatrix}
0 & 1 \\
1 & 1 \\
\end{pmatrix}
\begin{pmatrix}
f_{n} \\
f_{n+1} \\
\end{pmatrix}
$$

Обозначим за $A$ эту матрицу перехода. Чтобы посчитать $n$-ное число Фибоначчи, нужно применить $n$ раз эту матрицу к вектору $(f_0, f_1) = (0, 1)$.

Так как размер матрицы $A$ константный, её умножение саму на себя будет работать за $O(1)$. Значит, можно посчитать $A^n$ бинарным возведением в степень за $O(\log n)$ операций, домножить на вектор $(0, 1)$ и взять первый элемент результата — он будет равен в точности $f_n$, что мы и искали.

*Примечание.* Домножать на вектор явно даже не обязательно — если нам нужен только первый элемент, можно посмотреть на выражение для итогового результата и понять, что достаточно просто взять верхний правый элемент итоговой матрицы.

### Общий случай

В общем случае, когда мы учитываем не $2$, а $k$ последних элементов, причём с разными весами, линейная рекуррента $f_n = a_1 f_{n-1} + a_2 f_{n-2} + \ldots + a_k f_{n-k}$ имеет такую матрицу перехода:

$$
\begin{pmatrix}
0 & 1 & 0 & \ldots & 0 \\
0 & 0 & 1 & \ldots & 0 \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
0 & 0 & 0 & \ldots & 1 \\
a_k & a_{k-1} & a_{k-2} & \ldots & a_1 \\
\end{pmatrix}
$$

При домножении на вектор $(f_n, f_{n+1}, \ldots, f_{n+k-1})$ для последнего элемента получается в точности формула из определения, а все $(k-1)$ предыдущих просто перекопируются.

Отметим, что рекуррентные последовательности задаются не только коэффициентами $a_i$, но и начальными значениями $(f_1, f_2, \ldots, f_k)$.

**Упражнение.** Для линейной рекурренты порядка $k$ известны её коэффициенты $a_i$ и $k$ пар вида $(i, f_i)$. Восстановите первые $k$ элементов последовательности. 

### Геометрическая прогрессия

В [статье про бинарное возведение в степень](../binpow) мы обсуждали модификацию алгоритма для нахождения суммы геометрической прогрессии:

$$
f(n) = 1 + a + a^2 + \ldots + a^{n-1}
$$

Можно подойти к этой задаче с другой стороны. Заметим, что сумма первых $n$ степеней следующим образом выражается через сумму первых $(n-1)$ степеней:

$$
f(n) = f(n-1) \cdot a + 1
$$

Это почти линейная зависимость от предыдущего — только ещё добавляется неудобная константа. Можно применить следующий трюк: представить, что мы также зависим ещё и от $(n-2)$-го элемента, но положить его постоянно равным единице:

$$
\begin{pmatrix}
f_{n+1} \\
1 \\
\end{pmatrix}
=
\begin{pmatrix}
f_n \cdot a + 1 \\
0 + 1 \\
\end{pmatrix}
=
\begin{pmatrix}
a & 1 \\
0 & 1 \\
\end{pmatrix}
\begin{pmatrix}
f_{n} \\
1 \\
\end{pmatrix}
$$

Тогда эту матрицу можно аналогично возвести в степень $n$ и вернуть её правое верхнее поле.

## Динамическое программирование

Переходы в некоторых динамиках иногда тоже можно выразить в терминах матричного умножения.

**Задача.** Дана ориентированный граф $G$ размера $n < 500$. Требуется найти количество путей из $s$ в $t$ через ровно $k < 10^{18}$ переходов.

Если бы ограничение на $k$ было сильно меньше, мы бы ввели динамику $f_{i,v}$, равную количеству способов дойти до $v$-той вершины через ровно $i$ переходов. Пересчёт — перебор предпоследней вершины в пути:

$$
f_{i,v} = \sum_{u \in g_v} f_{i-1,u}
$$

Перепишем эту динамику в терминах матрицы смежности, в ячейке $G_{uv}$ которой содержится число ребер между $u$ и $v$ (ровно поэтому она называется матрицей).

$$
f_{i,v} = \sum_u f_{i-1,u} \cdot G_{uv}
$$

Выясняется, что вектор $f_{i}$ зависит от $f_{i-1}$ линейно: его $v$-тый элемент получается скалярным умножением вектора $f_{i-1}$ и $v$-того столбца матрицы смежности $G$. Значит, имеет место следующее равенство:

$$
f_i = G \cdot f_{i-1}
$$

Получается, $f_k$ можно раскрыть как $f_k = f_0 \cdot  G \cdot G \cdot \ldots \cdot G$ и применить бинарное возведение в степень.

По этой формуле нам потребуется $O(\log k)$ раз перемножить две матрицы размера $n \times n$, что суммарно будет работать за $O(n^3 \log k)$.

```c++
const int n;

matrix binpow(matrix a, int p) {
    // создадим единичную матрицу
    matrix b(n, vector<int>(n, 0));
    for (int i = 0; i < n; i++)
        b[i][i] = 1;

    while (p > 0) {
        if (p & 1)
            b = matmul(b, a);
         a = matmul(a, a);
         p >>= 1;
    }

    return b;
}
```

В получившейся матрице в ячейке $G_{ij}$ будет находиться количество способов дойти из $i$-той вершины в $j$-тую, используя ровно $k$ переходов. Ответом нужно просто вывести $G_{st}$.

### Модификации задачи

С небольшими изменениями этим методом можно решать много похожих задач:

- Практически не меняя сам алгоритм, можно решить задачу «с какой вероятностью мы попадём из вершины $s$ в вершину $t$», если вместо матрицы смежности даны вероятности, с которыми мы переходим из вершины в вершину в марковской цепи.
- Если нам не нужно количество способов, а только сам факт, можно ли дойти за ровно $k$ переходов, то можно обернуть матрицу в [битсеты](/cs/set-structures/bitset) и сильно ускорить решение.
- Если нас спрашивают «за не более, чем $k$ переходов», то вместо $k$-той степени матрицы мы можем вышеописанным методом посчитать сумму геометрической прогрессии. Альтернативно, можно для каждой вершины добавить вторую, в которую будет вести ребро из изначальной, а единственное исходящее ребро — это петля в саму себя.

Эту технику можно применить и к другим динамикам, где нужно посчитать количество способов что-то сделать — иногда очень неочевидными способами.

Например, можно решить такую задачу: найти количество строк длины $k \approx 10^{18}$, не содержащих данные маленькие запрещённые подстроки. Для этого нужно построить граф «легальных» переходов в [Ахо-Корасике](/cs/automata/aho-corasick), возвести его матрицу смежности в $k$-тую степень и просуммировать в нём первую строчку.

В некоторых изощрённых случаях в матричном умножении вместо умножения и сложения нужно использовать другие операции, которые ведут себя как умножение и сложение. Пример задачи: «найти путь от $s$ до $t$ с минимальным весом ребра, использующий ровно $k$ переходов»; здесь нужно возводить в $(k-1)$-ую степень матрицу весов графа, и вместо и сложения, и умножения использовать минимум из двух весов.

Также в контексте нахождения кратчайших путей известен «[distance product](https://en.wikipedia.org/wiki/Min-plus_matrix_multiplication)»:

$$
C_{ij} = \min_k \{ a_{ik} + b_{kj} \}
$$

Возводя матрицу весов в $k$-тую степень мы получаем, соответственно, минимальное расстояние от $a$ до $b$, используя не более $k$ переходов. В частности, при $k=n$ мы за $O(n^3 \log n)$ операций получим матрицу кратчайших расстояний между всеми парами вершины, хотя для этого есть и [более прямые методы](https://ru.wikipedia.org/wiki/%D0%90%D0%BB%D0%B3%D0%BE%D1%80%D0%B8%D1%82%D0%BC_%D0%A4%D0%BB%D0%BE%D0%B9%D0%B4%D0%B0_%E2%80%94_%D0%A3%D0%BE%D1%80%D1%88%D0%B5%D0%BB%D0%BB%D0%B0).

<!--

### Применение набора геометрических операций к точкам

Условие. Даны n точек p_i, и даны m преобразований, которые надо применить к каждой из этих точек. Каждое преобразование — это либо сдвиг на заданный вектор, либо масштабирование (умножение координат на заданные коэффициенты), либо вращение вокруг заданной оси на заданный угол. Кроме того, имеется составная операция циклического повторения: она имеет вид "повторить заданное число раз заданный список преобразований" (операции циклического повторения могут вкладываться друг в друга).

Требуется вычислить результат применения указанных операций ко всем точкам (эффективно, т.е. за время, меньшее чем O(n \cdot length), где length — общее количество операций, которые необходимо сделать).

Решение. Посмотрим на разные виды преобразований с точки зрения того, как они изменяют координаты:

Операция сдвига — она просто прибавляет ко всем координатам единицу, домноженную на некоторые константы.
Операция масштабирования — она умножает каждую координату на некоторую константу.
Операция вращения вокруг оси — её можно представить следующим образом: новые получаемые координаты можно записать как линейную комбинацию старых.
(Мы не будем здесь уточнять, каким образом это производится. Например, можно для простоты представить это в виде комбинации пяти двумерных поворотов: сначала в плоскостях OXY и OXZ так, чтобы ось вращения совпала с положительным направлением оси OX, затем требуемый поворот вокруг оси в плоскости YZ, затем обратные повороты в плоскостях OXZ и OXY так, чтобы ось вращения вернулась в своё исходное положение.)

Как легко видеть, каждое из этих преобразований — это пересчёт координат по линейным формулам. Таким образом, любое такое преобразование можно записать в виде матрицы 4 \times 4:

 \begin{pmatrix}
a_11 & a_{12} & a_{13} & a_{14} [...]

которое при умножении (слева) на строку из старых координат и константы-единицы даёт строку из новых координат и константы-единицы:

 \begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot[...]

(Почему понадобилось вводить фиктивную четвёртую координату, всегда равную единице? Без этого не получилось бы реализовать операцию сдвига: ведь сдвиг — это как раз прибавление к координатам единицы, домноженной на некоторые коэффициенты. Без фиктивной единицы мы бы смогли только реализовывать линейные комбинации самих координат, а прибавлять к ним заданные константы — не смогли бы.)

Теперь решение задачи становится почти тривиальным. Раз каждая элементарная операция описывается матрицей, то последовательность операций описывается произведением этих матриц, а операция циклического повторения — возведением этой матрицы в степень. Таким образом, мы за время O (m \cdot \log repetition) можем предпосчитать матрицу 4 \times 4, описывающую все преобразования, и затем просто умножить каждую точку p_i на эту матрицу — тем самым, мы ответим на все запросы за время O (n).

-->
