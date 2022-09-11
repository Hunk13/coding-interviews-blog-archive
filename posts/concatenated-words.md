Concatenated Words
==================

Mar 8, 2021 · 6 min read

Дан массив слов без дубликатов. Нужно найти среди них такие, которые могут быть составлены конкатенацией других слов (как минимум, двух).

![](/images/concatenated-words--ex.jpg)

Решение [#](#решение)
---------------------

По сути, нужно рекурсивно проверить строчку на наличие префиксов из словаря. По форме задача похожа [на поиск в глубину](/tags/dfs/).

    function canSplit(word, concatsCount = 0) {
      // базовый случай для рекурсии:
      // нет строки, то есть проверили все префиксы
      if (word.length === 0) {
        // считаем, что можно разбить,
        // если для этого сделали больше
        // чем одну конкатенацию,
        // (то есть конкатенацию самого
        // этого слова с пустой строкой)
        // а иначе это будет просто
        // исходное слово
        return concatsCount > 1;
      }
      for (let i = 0; i < words.length; i++) {
        if (
          // если нашли слово, которое является префиксом
          word.indexOf(words[i]) === 0 &&
          // рекурсивно поищем как разбить оставшуюся строку
          canSplit(word.substring(words[i].length), concatsCount + 1)
        ) {
          // если удалось найти — отлично, сразу вернём true,
          // если не удалось, то поиск должен продолжиться
          // в цикле выше
          return true;
        }
      }
      return false;
    }
    

В чем проблема данного решения? В скорости.

Во-первых, заметим, что нет смысла проверять на префиксы слова, которые больше в котором ищем. Префикс по определению будет меньше по длине. Соответственно, можно слова отсортировать по длине и обрывать цикл как только длина выходит за предел.

    for (let i = 0; i < words.length; i++) {
    + if (words[i].length > word.length) {
    +   break;
    + }
    }
    

Попробуем сдать — получаем stackoverflow 🤔

![](/images/concatenated-words--stackoverflow.jpg)

В условии сказано, что длина слов может быть `0 <= words[i].length <= 1000`, а значит пустая строка разрешена. В случае с пустой строкой `canSplit` зацикливается.

    if (
        // пустая строка всегда заматчится на начало,
        word.indexOf(words[i]) === 0 &&
        // а substring от нуля даёт ту же строку,
        // как следствие — бесконечный цикл
        canSplit(word.substring(words[i].length), concatsCount + 1)
    )
    

> ⚠️ Нужно проверять код на краевые случаи, то есть что будет при пустой строке, и длины 1000.

Кстати, это важный сигнал для интервьюера. Код нужно тестировать не только на нормальных примерах, но и для краевых случаев.

Правим баг:

    if (
    +   words[i].length > 0 &&
        word.indexOf(words[i]) === 0 &&
        canSplit(word.substring(words[i].length), concatsCount + 1)
    )
    

Сдаём и получаем TLE — то есть программа работает слишком долго.

![](/images/concatenated-words--tle.jpg)

Где «бутылочное горлышко»? Копирование строк? Да, но на самом деле, это не самый большой слон в этой комнате.

    function canSplit(curr, suffixStart = 0, slicesCount = 0) {
      if (words[curr].length === suffixStart) {
        return slicesCount > 1;
      }
      for (let i = 0; i < words.length; i++) {
        if (words[i].length > words[curr].length) {
          break;
        }
        if (
          words[i].length > 0 &&
          words[curr].indexOf(words[i], suffixStart) === suffixStart &&
          canSplit(curr, suffixStart + words[i].length, slicesCount + 1)
        ) {
          return true;
        }
      }
      return false;
    }
    

Переписать без копирования строк — само по себе неплохое упражнение. Однако, при попытке сдать получаем всё тот же TLE.

На самом деле, основная проблема — поиск подстроки, а именно:

    words[curr].indexOf(words[i], suffixStart) === suffixStart;
    

То есть мы пробегаемся по всем словам, и для каждого из них пытаемся заматчить префикс, что даёт `O(N * M)` сложность по времени, где `N` количество слов, а `M` длина слова. И это на каждый `canSplit`!

Здесь на помощь приходят **префиксные деревья** (trie).

Суть в том, что можно построить дерево из указанных слов таким образом, что префиксы сложатся в определённые пути в этом дереве.

![](/images/concatenated-words--trie.jpg)

Нам не придётся каждый раз бегать по всем словам в поиске префиксов — достаточно найти путь в этом дереве до узла `isLeaf` (то есть конца слова)!

Мне это представляется как есть огромный массив слов, и мы их сложили одно на другое, так что общие префиксы разных слов «растворились» в этом дереве.

На указанном пример: dog, cat, dogcatdog

![](/images/concatenated-words--ex.jpg)

Мы сперва найдем путь от корня до `g`, то есть слово `dog`. Далее продолжим поиск для суффикса `catdog`, снова от корня дерева. И так далее.

Найдется префикс или нет — зависит от того дойдём ли мы до узла `isLeaf` или придём в тупик, что означает в словаре не было слов с таким префиксом.

    /**
     * Класс для работы с префиксным деревом.
     * Содержим один публичный метод — добавить слово.
     */
    class Trie {
      root = new Node("");
      // добавляем указанное слово в дерево
      add(word) {
        let curr = this.root;
    
        for (let i = 0; i < word.length; i++) {
          curr = curr.addNext(word[i]);
        }
        curr.isLeaf = true;
      }
    }
    
    /**
     * Класс для работы с узлом.
     * Позволяет добавлять и искать по деткам.
     */
    class Node {
      isLeaf = false;
      children = new Array(26);
      constructor(val) {
        this.val = val;
      }
      // добавляем следующий узел
      addNext(val) {
        const idx = this._idx(val);
        if (!this.children[idx]) {
          this.children[idx] = new Node(val);
        }
        return this.children[idx];
      }
      // ищем указанный символ
      getNext(val) {
        return this.children[this._idx(val)];
      }
      _idx(val) {
        return val.charCodeAt(0) - "a".charCodeAt(0);
      }
    }
    

Тогда `canSplit` можно переписать следующим образом.

    function canSplit(curr, suffixStart = 0, slicesCount = 0) {
      if (words[curr].length == suffixStart) {
        return slicesCount > 1;
      }
      // начинаем поиск от корня дерева,
      // обратите внимание, что кейс с пустой
      // строкой автоматически покрывается:
      // корень дерева и есть пустая строка,
      // и мы начинаем поиск со следующих за ней
      // символах
      let node = trie.root;
    
      // ищем, начиная с указанного индекса
      for (let i = suffixStart; i < words[curr].length; i++) {
        // берем очередного кандидата
        node = node.getNext(words[curr][i]);
        // есть ли следующая буква среди префиксов?
        // если зашли в тупик — дальше искать смысла нет
        if (!node) {
          return false;
        }
        // isLeaf означает, что указанный
        // префикс был среди слов в словаре,
        // значит начиная отсюда надо поискать
        // разбиение для суффикса
        if (node.isLeaf && canSplit(curr, i + 1, slicesCount + 1)) {
          return true;
        }
      }
      return false;
    }
    

Теперь сложность данного решения ограничена только глубиной дерева, то есть самым длиным словом! Это намного быстрее чем дёргать `indexOf` для каждого слова из списка.

Всё вместе.

    /**
     * @param {string[]} words
     * @return {string[]}
     */
    var findAllConcatenatedWordsInADict = function(words) {
      const trie = new Trie();
    
      function canSplit(curr, suffixStart = 0, slicesCount = 0) {
        if (words[curr].length == suffixStart) {
          return slicesCount > 1;
        }
        let node = trie.root;
    
        for (let i = suffixStart; i < words[curr].length; i++) {
          node = node.getNext(words[curr][i]);
          if (!node) {
            return false;
          }
          if (
              node.isLeaf &&
              canSplit(curr, i + 1, slicesCount + 1)
            ) {
            return true;
          }
        }
        return false;
      }
      const result = [];
    
      words.sort((a, b) => a.length - b.length);
    
      for (let i = 0; i < words.length; i++) {
        // трюк с отсечением поиска
        // по заведомо невозможным префиксам
        // имеет смысл добавлять в дерево только слова,
        // которые короче проверяемого
        if (canSplit(i)) {
          result.push(words[i]);
        } else {
          trie.add(words[i]);
        }
      }
      return result;
    };
    
    class Trie {
      // ...
    }
    
    class Node {
      // ...
    }
    

![](/images/concatenated-words--result.jpg)

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.