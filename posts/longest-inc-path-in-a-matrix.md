Longest Increasing Path in a Matrix
===================================

Apr 12, 2021 · 5 min read

Дана матрица чисел.

Разрешены переходы между соседними клетками в четырех направлениях (лево, право, верх и низ). Надо найти _самый длинный путь_ (длину, не сам путь), где числа отсортированы по возрастанию (ну или убывания, смотря с какой стороны обходить).

![](https://assets.leetcode.com/uploads/2021/01/05/grid1.jpg)

В данном случае ответ 4.

**[Задача на LeetCode](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)**.

Решение [#](#решение)
---------------------

Чем-то эта задача напомнила мне [задачу про поиск слов](https://vitkarpov.me/posts/word-ladder/). В том смысле, что можно так же свести всё к построению графа, потому что заданы переходы между ячейками, и поиск пути. Однако здесь нужно искать путь не между двумя заданными узлами, а самый длинный путь между любыми двумя узлами.

Попробуем накидать скелет:

```js
let result = 1;
for (let y = 0; y < h; y++) {
  for (let x = 0; x < w; x++) {
    result = Math.max(result, getMaxPathLen(x, y));
  }
}
return result;
```
    

Зная как найти максимальную длину разрешённого пути начиная от `(x, y)`, можно проверить все клетки и выбрать максимальное значение.

Каким образом искать максимальный путь? Это обход в глубину, аналогично [задаче про острова](https://vitkarpov.me/posts/number-of-islands/).

Проверяем все четыре направления. Если переход есть — идём в рекурсию, увеличивая длину пройдённого пути. А когда рекурсия открутится назад до текущей клетки, нужно выбрать максимальное значение из всех направлений.

```js
function getMaxPathLen(x, y) {
  let result = 1;
  if (x + 1 < w && matrix[y][x + 1] > matrix[y][x]) {
    result = Math.max(result, 1 + getMaxPathLen(x + 1, y));
  }
  if (x - 1 >= 0 && matrix[y][x - 1] > matrix[y][x]) {
    result = Math.max(result, 1 + getMaxPathLen(x - 1, y));
  }
  if (y + 1 < h && matrix[y + 1][x] > matrix[y][x]) {
    result = Math.max(result, 1 + getMaxPathLen(x, y + 1));
  }
  if (y - 1 >= 0 && matrix[y - 1][x] > matrix[y][x]) {
    result = Math.max(result, 1 + getMaxPathLen(x, y - 1));
  }
  return result;
}
``` 

В рекурсии важно вовремя остановиться. Есть ли сейчас возможность прогуляться по кругу, попасть на уже пройдённую клетку, и зациклиться? Обычно это решается следующим образом.

```diff
function getMaxPathLen(x, y) {
+   const k = key(x, y);
+   if (visited.has(k)) {
+       return -Infinity;
+   }
+   visited.add(k);
    let result = 1;
    if (x + 1 < w && matrix[y][x + 1] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x + 1, y));
    }
    if (x - 1 >= 0 && matrix[y][x - 1] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x - 1, y));
    }
    if (y + 1 < h && matrix[y + 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x, y + 1));
    }
    if (y - 1 >= 0 && matrix[y - 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x, y - 1));
    }
+   visited.delete(k);
    return result;
}
```

Заводим сет, в котором будем запоминать пройденные клетки _на данном пути_. Когда рекурсия открутится назад до текущей клетки, не забудем «освободить клетку», чтобы она смогла стать частью другого пути.

Однако, в данном случае такой ситуации быть не может. Потому что мы ищем строго возрастающий путь, то есть закольцеваться он никак не сможет, поэтому дополнительная проверка не нужна.

Какая сложность?

![](/images/longest-inc-path-in-a-matrix--2n.jpg)

Если правильно подобрать значения, то путь в каждой клетке может ветвиться на ещё два (как минимум), поэтому `O(2^N)`, где `N` — количество клеток.

Учитывая, что такую историю нужно вызвать для каждой клетки, получится `O(N * 2^N)`.

Для небольшой матрицы оно, конечно, будет работать. Однако, у нас ограничения `1 <= m, n <= 200`, то есть `1 <= N <= 40000`. Многовато для степени двойки.

Время для оптимизаций! Где лишняя работа?

Допустим, что мы вычислили максимальное значение пути для определённой клетки, а потом пришли в неё снова в рамках другого пути. Надо ли снова идти рекурсивно от этой клетки? Нет. Потому дальше мы просто зря проверим тонну хвостов, которые дадут ровно тот же результат — «максимальнее» оставшайся часть пути уже не станет, если до этого была вычислена правильно.

В этом и есть суть дпшечки или мемоизации здесь. Сложность при этом уменьшится до `O(N)`, потому что количество состояний, которое надо кешировать, ограничено количеством клеток в матрице.

```diff
function getMaxPathLen(x, y) {
+   const k = key(x, y);
+   if (dp.has(k)) {
+     return dp.get(k);
+   }
    let result = 1;
    if (x + 1 < w && matrix[y][x + 1] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x + 1, y));
    }
    if (x - 1 >= 0 && matrix[y][x - 1] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x - 1, y));
    }
    if (y + 1 < h && matrix[y + 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x, y + 1));
    }
    if (y - 1 >= 0 && matrix[y - 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + getMaxPathLen(x, y - 1));
    }
+   dp.set(k, result);
    return result;
}
```

Когда полностью проверили все пути с началом в данном клетке — записали. Потом, при проверке другого пути — прочитали. Таким образом, отсекаем тонну лишних пересчётов, которые привели бы к тому же самому ответу.

Внимательный читатель может спросить что такое `key`?

Функция, которая просто вычисляет ключ в хеш-таблицы для ячейки, по которому будем кешировать значения. В данном случае, реализация у нее очень простая:

```js
function key(x, y) {
  return y * matrix[0].length + x;
}
```

Ячейки матрицы легко мапятся на индексы массива. С такой «хеш-таблицей» можно было б и просто массив завести 😊

Всё вместе.

```js
/**
 * @param {number[][]} matrix
 * @return {number}
 */
var longestIncreasingPath = function(matrix) {
  const h = matrix.length;
  const w = matrix[0].length;
  const dp = new Map();
  function key(x, y) {
    return y * w + x;
  }
  function dfs(x, y) {
    const k = key(x, y);
    if (dp.has(k)) {
      return dp.get(k);
    }
    let result = 1;
    if (x + 1 < w && matrix[y][x + 1] > matrix[y][x]) {
      result = Math.max(result, 1 + dfs(x + 1, y));
    }
    if (x - 1 >= 0 && matrix[y][x - 1] > matrix[y][x]) {
      result = Math.max(result, 1 + dfs(x - 1, y));
    }
    if (y + 1 < h && matrix[y + 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + dfs(x, y + 1));
    }
    if (y - 1 >= 0 && matrix[y - 1][x] > matrix[y][x]) {
      result = Math.max(result, 1 + dfs(x, y - 1));
    }
    dp.set(k, result);
    return result;
  }
  let result = 1;
  for (let y = 0; y < h; y++) {
    for (let x = 0; x < w; x++) {
      result = Math.max(result, dfs(x, y));
    }
  }
  return result;
};
```

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.