---
tags: linux2022
---

# 2022q1 Homework1 (lab0)
contributed by < [blue76815](https://github.com/blue76815/lab0-c) >

## 實驗環境
```shell
$ gcc --version
gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0

$ lscpu
架構：				86_64
CPU 作業模式：			32-bit, 64-bit
Byte Order:			Little Endian
Address sizes:			39 bits physical, 48 bits virtual
CPU(s):				12
On-line CPU(s) list:		0-11
每核心執行緒數：			2
每通訊端核心數：			6
Socket(s):			1
NUMA 節點：			1
供應商識別號：			GenuineIntel
CPU 家族：			6
型號：				158
Model name:			Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
製程：				10
CPU MHz：			900.049
CPU max MHz:			4100.0000
CPU min MHz:			800.0000
BogoMIPS:			4399.99
虛擬：				VT-x
L1d 快取：			192 KiB
L1i 快取：			192 KiB
L2 快取：			1.5 MiB
L3 快取：			9 MiB
NUMA node0 CPU(s)：		0-11
```

## [作業要求](https://hackmd.io/@sysprog/linux2022-lab0#-%E4%BD%9C%E6%A5%AD%E8%A6%81%E6%B1%82)
:::spoiler 清單
 - [x] 在 GitHub 上 fork [lab0-c](https://github.com/sysprog21/lab0-c)
- [x] 依據上述指示著手修改 `queue.[ch]` 和連帶的檔案，測試後用 Git 管理各項修改，要能滿足 `$ make test` 自動評分系統的所有項目。
    - [x] 在提交程式變更前，務必詳閱 [如何寫好 Git Commit Message](https://blog.louie.lu/2017/03/21/%E5%A6%82%E4%BD%95%E5%AF%AB%E4%B8%80%E5%80%8B-git-commit-message/)
    - [x] 詳閱「[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list)」並應用裡頭提及的手法，例如 pointer to pointer (指向某個指標的指標，或說 indirect pointer)，讓程式碼更精簡且有效
    - [x] 用鏈結串列 ([linked list](https://en.wikipedia.org/wiki/Linked_list)) 來實作佇列 ([queue](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)))，請詳閱「[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list)」，以得知 Linux 核心原始程式碼風格的鏈結串列實作的考量，並研讀 [The Linux Kernel API - List Management Functions](https://www.kernel.org/doc/html/latest/core-api/kernel-api.html)。
    - [ ] 研讀 Linux 核心原始程式碼的 [lib/list_sort.c](https://github.com/torvalds/linux/blob/master/lib/list_sort.c) 並嘗試引入到 `lab0-c` 專案，比較你自己實作的 merge sort 和 Linux 核心程式碼之間效能落差
- [ ] 在 `qtest` 提供新的命令 `shuffle`，允許藉由 [Fisher–Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle) 演算法，對佇列中所有節點進行洗牌 (shuffle) 操作
- [ ] 在 `qtest` 提供新的命令 `web`，提供 web 伺服器功能，注意: web 伺服器運作過程中，`qtest` 仍可接受其他命令
    - [ ] 可依據前述說明，嘗試整合 [tiny-web-server](https://github.com/7890/tiny-web-server) 
- [x] 除了修改程式，也要編輯「[作業區](https://hackmd.io/@sysprog/linux2022-homework1)」，增添開發紀錄和 GitHub 連結，除了提及你如何逐步達到自動評分程式的要求外，共筆也要涵蓋以下:
    - [ ] 開啟 [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)，修正 `qtest` 執行過程中的錯誤
        - [ ] 先執行 `qtest` 再於命令提示列輸入 `help` 命令，會使 開啟 [Address Sanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) 觸發錯誤，應予以排除
    - [ ] 運用 Valgrind 排除 `qtest` 實作的記憶體錯誤，並透過 Massif 視覺化 "simulation" 過程中的記憶體使用量，需要設計對應的實驗
    - [ ] 解釋 [select](http://man7.org/linux/man-pages/man2/select.2.html) 系統呼叫在本程式的使用方式，並分析 [console.c](https://github.com/sysprog21/lab0-c/blob/master/console.c) 的實作，說明其中運用 CS:APP [RIO 套件](http://csapp.cs.cmu.edu/2e/ch10-preview.pdf) 的原理和考量點。可對照參考 [CS:APP 第 10 章重點提示](https://hackmd.io/@sysprog/H1TtmVTTz)
    - [ ] 研讀論文〈[Dude, is my code constant time?](https://eprint.iacr.org/2016/1123.pdf)〉，解釋本程式的 "simulation" 模式是如何透過以實驗而非理論分析，達到驗證時間複雜度，需要解釋 [Student's t-distribution](https://en.wikipedia.org/wiki/Student%27s_t-distribution) 及程式實作的原理。注意：現有實作存在若干致命缺陷，請討論並提出解決方案;
- [ ] 指出現有程式的缺陷 (提示: 和 [RIO 套件](http://csapp.cs.cmu.edu/2e/ch10-preview.pdf) 有關) 或者可改進之處 (例如更全面地使用 Linux 核心風格的鏈結串列 API)，嘗試實作並提交 pull request
:::
## `queue.c` 實作
* `q_new`: 建立新的「空」佇列;
* `q_free`: 釋放佇列所佔用的記憶體;
* `q_insert_head`: 在佇列開頭 (head) 插入 (insert) 給定的新節點 (以 LIFO 準則);
* `q_insert_tail`: 在佇列尾端 (tail) 插入 (insert) 給定的新節點 (以 FIFO 準則);
* `q_remove_head`: 在佇列開頭 (head) 移去 (remove) 給定的節點;
* `q_release_element`: 釋放特定節點的記憶體空間;
* `q_size`: 計算目前佇列的節點總量;
* `q_delete_mid`: 移走佇列中間的節點，詳見 [LeetCode 2095. Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/)
* `q_delete_dup`: 在已經排序的狀況，移走佇列中具備重複內容的節點，詳見 [LeetCode 82. Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/)
* `q_swap`: 交換佇列中鄰近的節點，詳見 [LeetCode 24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)
* `q_reverse`: 以反向順序重新排列鏈結串列，該函式不該配置或釋放任何鏈結串列節點，換言之，它只能重排已經存在的節點;
* `q_sort`: 以==遞增順序==來排序鏈結串列的節點，可參閱 [Linked List Sort](https://npes87184.github.io/2015-09-12-linkedListSort/) 得知實作手法;

pointer to pointer 參考 [Linked lists, pointer tricks and good taste](https://github.com/mkirchner/linked-list-good-taste) 寫法

實做 `queue.c` 時參閱
* [linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h)
* [The Linux Kernel API](https://www.kernel.org/doc/html/latest/core-api/kernel-api.html)
* [内核双链表遍历：带 safe 和不带 safe 的区别](https://biscuitos.github.io/blog/LIST_ADV_safe/)
* [Bidirect-list](https://biscuitos.github.io/tags#Bidirect-list)
* 你所不知道的 C 語言: linked list 和非連續記憶體
	* [Linked list 在 Linux 核心原始程式碼](https://hackmd.io/@sysprog/c-linked-list?type=view#Linked-list-%E5%9C%A8-Linux-%E6%A0%B8%E5%BF%83%E5%8E%9F%E5%A7%8B%E7%A8%8B%E5%BC%8F%E7%A2%BC)

### q_new
此操作為建立新的「空」佇列，因此是建立 `struct list_head head` 頭端，而不是新建一個 element_t 節點
1. 配置一個新 head
2. 若配置失敗則回傳 NULL
3. 調用 [The Linux Kernel API](https://www.kernel.org/doc/html/latest/core-api/kernel-api.html) `INIT_LIST_HEAD()`初始化head

:::danger
注意書寫規範：中英文間用一個半形空白字元區隔
:notes: jserv
:::
:::info
已修正
:::
```c
struct list_head *q_new()
{
    struct list_head *head = malloc(sizeof(struct list_head));
    if( head == NULL) 
        return NULL;
    INIT_LIST_HEAD(head);
    return head;
}
```
### q_free
1. 調用 `list_for_each_entry_safe` 尋訪 `struct list_head` 環狀佇列
2. `list_del()` 只是將 `node` 從其原本的 linked list 連接移除
3. 還得用 `q_release_element(node)` 才能將節點記憶體釋放
```c
void q_free(struct list_head *l)
{
    if (l == NULL)
        return;

    element_t *node = NULL, *tmp_node = NULL;
    list_for_each_entry_safe (node, tmp_node, l, list) {
        list_del(&node->list);
        q_release_element(node);
    }
    free(l);
}
```
### q_insert_head
1. `malloc `配置新節點
2. 將字串資料複製到 `node->value`
3. `list_add()` 將指定的 node 插入到 linked list `struct list_head head` 的開頭
```c
bool q_insert_head(struct list_head *head, char *s)
{
    if (s == NULL || head == NULL)
        return false;
    element_t *node = malloc(sizeof(element_t));
    if (node == NULL)
        return false;

    node->value = (char *) malloc(sizeof(char) * strlen(s) + 1);
    if (node->value == NULL) {
        q_release_element(node);
        return false;
    }
    memset(node->value, '\0', sizeof(char) * strlen(s) + 1);
    memcpy(node->value, s, sizeof(char) * strlen(s));
    list_add(&node->list, head);
    return true;
}
```
#### 注意事項
1. 節點插入`data` 資料時，得用 `element_t` 才能配置 
```c
typedef struct {
    /* Pointer to array holding string.
     * This array needs to be explicitly allocated and freed
     */
    char *value;
    struct list_head list;
} element_t;
```
圖片來源：[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list?type=view#Linked-list-%E5%9C%A8-Linux-%E6%A0%B8%E5%BF%83%E5%8E%9F%E5%A7%8B%E7%A8%8B%E5%BC%8F%E7%A2%BC)
![](https://i.imgur.com/TIxPBGA.png)

2. 要訪問 linked list 各別節點內的資料，可調用 
```
list_entry(node, type, member)
list_first_entry(head, type, member)
list_last_entry(head, type, member)
```
3. 配置 `strlen(s) + 1` 是因為
[Linux Programming Interface 查詢 strlen](https://man7.org/linux/man-pages/man3/strlen.3.html) 有提到
> ### DESCRIPTION
> The strlen() function calculates the length of the string pointed to by s, <span class="red">**excluding the terminating null byte ('\0').**</span>
> 
> ### RETURN VALUE
> The strlen() function returns the number of bytes in the string pointed to by s.

`strlen()` 計算字元個數到 **null byte ('\0')** 就停止，因此字數不包含 **null byte ('\0')**。
多加一格，是為了**在字串結尾多填一格 NULL 值**，
**避免 `strlen(const char str)` 計算字串長度時，把字尾相鄰的記憶體殘值也列入字元個數**

### q_insert_tail
1. `malloc `配置新節點
2. 將字串資料複製到 `node->value`
3. 差別只在插入 link-list 時，得用 `list_add_tail(&node->list, head);` 插入到尾端
```c
bool q_insert_tail(struct list_head *head, char *s)
{
    if (head == NULL || s == NULL)
        return false;

    element_t *node = malloc(sizeof(element_t));

    if (node == NULL)
        return false;


    node->value = (char *) malloc(sizeof(char) * strlen(s) + 1);
    if (node->value == NULL) {
        q_release_element(node);
        return false;
    }
    memset(node->value, '\0', sizeof(char) * strlen(s) + 1);
    memcpy(node->value, s, sizeof(char) * strlen(s));
    list_add_tail(&node->list, head);
    return true;
}
```

### q_remove_head
* 調用 `list_first_entry()` ，取得 linked list 的開頭
```c
element_t *q_remove_head(struct list_head *head, char *sp, size_t bufsize)
{
    if (head == NULL || list_empty(head))
        return NULL;

    element_t *node = list_first_entry(head, element_t, list);

    list_del_init(&(node->list));

    if (sp) {
        size_t length = strlen(node->value) + 1;
        length = length > bufsize ? bufsize : length;
        memset(sp, '\0', length);
        memcpy(sp, node->value, length - 1);
    }
    return node;
}
```


### q_remove_tail
* 調用 `list_last_entry()` ，取得 linked list 的結尾

```c
element_t *q_remove_tail(struct list_head *head, char *sp, size_t bufsize)
{
    if (head == NULL || list_empty(head))
        return NULL;

    element_t *node = list_last_entry(head, element_t, list);

    list_del_init(&(node->list));

    if (sp) {
        size_t length = strlen(node->value) + 1;
        length = length > bufsize ? bufsize : length;
        memset(sp, '\0', length);
        memcpy(sp, node->value, length - 1);
    }

    return node;
}
```

### q_release_element
```c
void q_release_element(element_t *e)
{
    free(e->value);
    free(e);
}
```

### q_size
1. 調用 `list_for_each_entry_safe` 
2. 尋訪 `struct list_head` 環狀佇列計次

```c
int q_size(struct list_head *head)
{
    if (head == NULL || list_empty(head))
        return 0;

    int size = 0;
    struct list_head *node = NULL;

    list_for_each (node, head) {
        size++;
    }
    return size;
}
```

### q_delete_mid
#### 第ㄧ版 
1. 直接用 `q_size(head)` 取得佇列長度
2. 除以2就能求出長度一半，可作為計算要移動節點幾次才到達 `mid_head`
3. 到達 `mid_head` 後，用 `list_entry()` 取得節點內容刪除
```c
bool q_delete_mid(struct list_head *head)
{
    // https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/
    if (head == NULL || list_empty(head))
        return false;
    int mid_length = q_size(head) / 2;
    struct list_head *mid_head = head->next;
    while (mid_length > 0) {
        mid_head = mid_head->next;
        mid_length--;
    }

    element_t *ele_node = list_entry(mid_head, element_t, list);
    list_del(mid_head);
    free(ele_node->value);
    free(ele_node);
    return true;
}
```
#### 第二版
參考[金刀的算法小册子 0876-链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/solution/jin-dao-ji-shu-fa-kuai-man-zhi-zhen-yyds-9fgd/)
改成快慢兩種指標做 Floyd’s Cycle detection
```c
bool q_delete_mid(struct list_head *head)
{
    // https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/
    if (head == NULL || list_empty(head))
        return false;
    
    struct list_head *fast = head->next;
    struct list_head *slow = head->next;
    while (fast!=head && fast->next !=head) {
        slow = slow->next;
        fast = fast->next->next;
    }

    element_t *ele_node = list_entry(slow, element_t, list);
    list_del(slow);
    free(ele_node->value);
    free(ele_node);
    return true;
}
```


### q_delete_dup
參考 [SmallHanley](https://hackmd.io/@SmallHanley/linux2022-lab0#q_delete_dup) 同學的方法

1. 檢查 `head` 是否為 `NULL`，是則回傳 `false`。
1. 用一個 bool 值 `last_same_node` 判斷相鄰重複的節點，是否已經刪除到最後一個。
1. 呼叫 `list_for_each_safe (node, tmp, head)` 走訪每個節點
1. 走訪節點時，此`node`內是儲存**某 element_t 節點的記憶體位址**，得用 `list_entry(node, element_t, list)` 和 `list_entry(node, element_t, list)` 才能解析出 element_t 節點內的 value 成員
1. 判斷此 element_t  節點和下一個 element_t 節點的 `value` 是否相同，是則將 `last_same_node` 設為 `true`，然後從串列中移除此節點並釋放相關空間。
1. 若 `value` 不同，而 `last_same_node` 依然是 `true`，代表此節點是相鄰重複的節點中的最後一個，將 `last_same_node` 設回 `false` 並釋放相關空間。
例如 link-list 1==>1==>1==>2==>3 
當刪除到剩 1==>2==>3 時，此 1 節點就是**相鄰重複節點中的最後一個**

[list_entry 介紹](https://biscuitos.github.io/blog/LIST_list_entry/)
[内核双链表遍历：带 safe 和不带 safe 的区别](https://biscuitos.github.io/blog/LIST_ADV_safe/)
> ```
> #define list_for_each(pos, head) \
>         for (pos = (head)->next; pos != (head); pos = pos->next)
> 
> 
> #define list_for_each_safe(pos, n, head) \
>         for (pos = (head)->next, n = pos->next; pos != (head); \
>                 pos = n, n = pos->next)
> ```
> 从两个接口的定义差别只看出，**list_for_each_safe() 的定义多了一个参数 n，这个参数 n 用于缓存下一个节点**，其余并没有什么逻辑上的差异。
> 
所以寫 `list_for_each_safe (node, tmp, head) `
相當於用 `node` 節點 跑 `for` 迴圈

```c
bool q_delete_dup(struct list_head *head)
{
    // https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/
    if (head == NULL || list_empty(head))
        return false;
    struct list_head *node, *tmp;
    bool last_same_node = false;
    list_for_each_safe (node, tmp, head) {
        element_t *curr_element_t = list_entry(node, element_t, list);
        element_t *next_element_t = list_entry(node->next, element_t, list);
        if (node->next != head &&
            strcmp(curr_element_t->value, next_element_t->value) == 0) {
            last_same_node = true;
            list_del(node);
            free(curr_element_t->value);
            free(curr_element_t);
        } else if (last_same_node == true) {
            last_same_node = false;
            list_del(node);
            free(curr_element_t->value);
            free(curr_element_t);
        }
    }
    return true;
}
```

### q_swap
在[官方 linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h) 中，發現有 `list_swap()` 可調用
```c
/**
 * list_swap - replace entry1 with entry2 and re-add entry1 at entry2's position
 * @entry1: the location to place entry2
 * @entry2: the location to place entry1
 */
static inline void list_swap(struct list_head *entry1,
			     struct list_head *entry2)
{
	struct list_head *pos = entry2->prev;

	list_del(entry2);
	list_replace(entry1, entry2);
	if (pos == entry1)
		pos = entry2;
	list_add(entry1, pos);
}
```
調用 `list_swap()` 編譯時出現
```
$ make
  CC    queue.o
queue.c: In function ‘q_swap’:
queue.c:267:9: error: implicit declaration of function ‘list_swap’ [-Werror=implicit-function-declaration]
  267 |         list_swap(node,node->next);
      |         ^~~~~~~~~
cc1: all warnings being treated as errors
make: *** [Makefile:50：queue.o] 錯誤 1
```
發現我們 `lab0` 專案內的 `list.h` 中
沒有加入[官方 linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h) 的 `list_swap()` 程式碼

於是我自己將官方 `list_swap()` 程式碼加在 `lab0` 專案內的 `queue.c` 中
再次編譯
```
$ make
  CC    qtest.o
In file included from qtest.c:16:
list.h: In function ‘list_swap’:
list.h:348:2: error: implicit declaration of function ‘list_replace’; did you mean ‘list_splice’? [-Werror=implicit-function-declaration]
  348 |  list_replace(entry1, entry2);
      |  ^~~~~~~~~~~~
      |  list_splice
cc1: all warnings being treated as errors
make: *** [Makefile:50：qtest.o] 錯誤 1
```

`queue.c` 中還要再補上 [官方 linux/list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h) `list_replace()`
```c
/**
 * list_replace - replace old entry by new one
 * @old : the element to be replaced
 * @new : the new element to insert
 *
 * If @old was empty, it will be overwritten.
 */
static inline void list_replace(struct list_head *old,
				struct list_head *new)
{
	new->next = old->next;
	new->next->prev = new;
	new->prev = old->prev;
	new->prev->next = new;
}
```
編譯成功


```c
void q_swap(struct list_head *head)
{
    // https://leetcode.com/problems/swap-nodes-in-pairs/
    if ( head==NULL|| list_empty(head))
        return;

    struct list_head *node = head->next, *tmp_node;
    while (node != head && node->next != head) {
        tmp_node = node->next->next;//相鄰兩節點交換前，先備份第3個節點位址到 tmp_node 變數中
        list_swap(node, node->next);//相鄰兩節點交換
        node = tmp_node;//載入新的節點
    }
}
```
### q_reverse
將 `prev` 與 `next` 兩個指標的內容交換(可藉由 `tmp_node` 暫存)

1. `struct list_head` 為環狀佇列，當節點走訪一圈後就會回到 `head` 位址，利用此特點判斷 `while(node!=head)` 是否走訪一圈回到`head` 	
	
圖片來源：[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list?type=view#Linked-list-%E5%9C%A8-Linux-%E6%A0%B8%E5%BF%83%E5%8E%9F%E5%A7%8B%E7%A8%8B%E5%BC%8F%E7%A2%BC)
![](https://i.imgur.com/TIxPBGA.png)

2. 先做 `next_node=node->next` ，備份好下個節點
3. 每個節點逐一將 `prev` 與 `next` 兩個指標內儲存的地址交換(藉由 `tmp_node` 變數，暫存節點地址)
4. 離開前，記得 `head` 節點的 `prev` 與 `next` 也要交換
```c
void q_reverse(struct list_head *head) {
    if ( head==NULL || list_empty(head)) {
        return;
    } 
    struct list_head *node=head->next,*next_node,*tmp_node;
    while(node!=head)//判斷是否走訪一圈回到 head
    {
	/* prev 與 next 交換前，先備份好下個節點*/	
       next_node=node->next;
		
       /*將 prev 與 next 兩個指標內儲存的地址交換*/	
       tmp_node=node->next;
       node->next=node->prev;
       node->prev = tmp_node; 
		
	/* 更新下一個節點*/
       node=next_node;
    } 
    /*離開前 記得 head 節點的 prev 與 next 也要交換*/
    tmp_node = head->next;
    head->next = head->prev;
    head->prev = tmp_node;      
}
```
### q_sort（遞迴版）
參閱
* [你所不知道的 C 語言: linked list 和非連續記憶體 Merge Sort 的實作](https://hackmd.io/@sysprog/c-linked-list#Merge-Sort-%E7%9A%84%E5%AF%A6%E4%BD%9C) 遞迴作法
* [freshLiver](https://hackmd.io/@freshLiver/2022q1-hw1#%E5%AF%A6%E4%BD%9C-queue-%E7%9B%B8%E9%97%9C%E5%87%BD%E5%BC%8F) 同學
* [kevinshieh0225](https://hackmd.io/@Masamaloka/2022q1-hw1) 同學

```c
void q_sort(struct list_head *head)
{
    if (head == NULL || list_empty(head)) {
        return;
    }
    head->prev->next = NULL;
    head->next = mergesort_list(head->next);
    struct list_head *ptr = head;
    while (ptr->next != NULL) {
        ptr->next->prev = ptr;
        ptr = ptr->next;
    }
    ptr->next = head;
    head->prev = ptr;
}
```
`mergesort_list()` 的詳細實做如下
```c
struct list_head *mergeTwoLists(struct list_head *L1, struct list_head *L2)
{
    struct list_head *head = NULL;
    struct list_head **ptr = &head;
    for (; L1 && L2; ptr = &(*ptr)->next) {
        element_t *L1_element_t = list_entry(L1, element_t, list);
        element_t *L2_element_t = list_entry(L2, element_t, list);
        if (strcmp(L1_element_t->value, L2_element_t->value) <= 0) {
            *ptr = L1;
            L1 = L1->next;
        } else {
            *ptr = L2;
            L2 = L2->next;
        }
    }
    *ptr = (struct list_head *) ((uintptr_t) L1 | (uintptr_t) L2);
    return head;
}

static struct list_head *mergesort_list(struct list_head *head)
{
    if (!head || !head->next)
        return head;

    struct list_head *slow = head;
    for (struct list_head *fast = head->next; fast && fast->next;
         fast = fast->next->next)
        slow = slow->next;
    struct list_head *mid = slow->next;
    slow->next = NULL;

    struct list_head *left = mergesort_list(head), *right = mergesort_list(mid);
    return mergeTwoLists(left, right);
}
```
## 運用 Valgrind 排除 `qtest` 實作的記憶體錯誤
:::info
### valgrind 產生 massif 方法
以我們 lab0 內附的 traces/trace-15-perf.cmd 測試項為例

step1.產生 massif 方法

`$ valgrind --tool=massif ./qtest -f traces/trace-15-perf.cmd`

生成一個 massif.out.50141 檔
![](https://i.imgur.com/fqJEFZ9.png)
step2.印出massif 檔

例如我生成 massif.out.50141 檔
`$ ms_print massif.out.5014`
![](https://i.imgur.com/bQUAwl9.png)
### Massif-Visualizer 安裝方法
[valgrind massif內存分析](https://www.twblogs.net/a/5b8e7ae42b71771883457f9d)
或到我的 ubuntu 20.4 的 ubuntu software

![](https://i.imgur.com/N9T4GzQ.png)

搜尋 Massif Visualizer 
![](https://i.imgur.com/osdPaVP.png)
![](https://i.imgur.com/EFY2oFD.png)
我下載中英文兩套界面版本的 Massif Visualizer
![](https://i.imgur.com/985tdMu.png)

![](https://i.imgur.com/SUBNFbQ.png)
![](https://i.imgur.com/xpnaXq4.png)
![](https://i.imgur.com/Q1W9kpf.png)

![](https://i.imgur.com/omiIxPp.png)
:::
