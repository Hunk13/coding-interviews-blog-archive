Validate Binary Search Tree
===========================

Mar 15, 2021 · 3 min read

Определить является ли указанное дерево двоичным деревом поиска.

По определинию, это такое дерево, что:

*   любое _левое_ поддерево содержит узлы со значениями _меньше_ своего корня;
*   любое _правое_ поддерево содержит узлы со значениями _больше_ своего корня;
*   это должно быть верно для любого узла в дереве.

Пример:

![](/images/validate-binary-search-tree--ex.jpg)

Дерево не является валидным, потому что в правом поддереве от корня обнаружился узел с меньшим значением, а должны быть все строго больше.

[Задача на LeetCode](https://leetcode.com/problems/validate-binary-search-tree/).

Решение [#](#решение)
---------------------

Из условия сразу бросается в глаза рекурсивная природа задачи.

По определению любой узел должен быть сам по себе валидными деревом поиска, а значит решение будет выглядить как-то так:

    var isValidBST = function(root) {
      // ... какой-то базовый случай
      return isValidBST(root.left) && isValidBST(root.right);
    };
    

Базовый случай — тривиальный, то есть отсутствие узла считаем валидным деревом. И далее начинаем разматывать стек рекурсии наверх.

Как понять, что дерево перестало быть валидным? Нарушился инвариант. А именно для _любого узла_ его детки должны находиться в определённом интервале. Классическая ошибка — проверять только непосредственных детей на попадание в нужный интервал.

Кажется, вырисовывается интерфейс.

    function helper(root, min, max) {
      if (!root) {
        return true;
      }
      if (min < root.val && root.val < max) {
        return (
          helper(root.left, min, root.val) && helper(root.right, root.val, max)
        );
      }
      return false;
    }
    

По сути, получается поиск в глубину. На каждом шаге мы меняем интервал (min, max). Если идём в левое поддерево — больше не можем встретить узлы с большими значениями чем корень, то есть меняем max. Аналогично и для правого поддерева, то есть меняем min.

    /**
     * Definition for a binary tree node.
     * function TreeNode(val) {
     *     this.val = val;
     *     this.left = this.right = null;
     * }
     */
    /**
     * @param {TreeNode} root
     * @return {boolean}
     */
    var isValidBST = function(root) {
      function helper(root, min, max) {
        // базовый случай
        if (!root) {
          return true;
        }
        // идём дальше только
        // если выполняется инвариант
        if (min < root.val && root.val < max) {
          return (
            helper(root.left, min, root.val) && helper(root.right, root.val, max)
          );
        }
        return false;
      }
      // запустим с максимально
      // широким интервалом,
      // чтобы гарантированно
      // начать поиск для любого корня
      return helper(root, -Infinity, Infinity);
    };
    

У этой задачи есть и другое, довольно любопытное, решение. Попробуем сперва напечатать значения узлов, обходя дерево следующим образом.

    function inorder(root) {
      if (!root) {
        return;
      }
      inorder(root.left);
      console.log(root.val);
      inorder(root.right);
    }
    

Значения будут выведены в отсортированном виде. Это не случайно. Такой вариант обхода: посетить левый узел, напечатать, посетить правый узел — inorder, как раз гарантированно даёт сортировку для валидного дерева поиска.

Если это свойство нарушается — значит дерево не было валидным, что как раз и даёт решение.

    /**
     * Definition for a binary tree node.
     * function TreeNode(val) {
     *     this.val = val;
     *     this.left = this.right = null;
     * }
     */
    /**
     * @param {TreeNode} root
     * @return {boolean}
     */
    var isValidBST = function(root) {
      let prev = -Infinity;
      function inorder(root) {
        // базовый случай не меняется
        if (!root) {
          return true;
        }
        // inorder вариант обхода значит:
        // - идём налево
        // - что-то делаем (проверяем инвариант)
        // - идём направо
        if (!inorder(root.left)) {
          return false;
        }
        // инвариант нарушен
        if (root.val <= prev) {
          return false;
        }
        // если смогли здесь оказаться,
        // значит все «предыдущие узлы»
        // были проверены, и пора обновляться
        prev = root.val;
        return inorder(root.right);
      }
      return inorder(root);
    };
    

> [Подробнее](https://www.geeksforgeeks.org/tree-traversals-inorder-preorder-and-postorder/) про различные варианты обхода дерева и зачем они нужны.

PS. Обсудить можно в [телеграм-чате](https://t.me/ctci_chat_ru) любознательных программистов. Welcome! 🤗

Подписывайтесь на [мой твитер](https://twitter.com/vitkarpov) или [канал в телеграме](https://t.me/coding_interviews), чтобы узнавать о новых разборах задач.