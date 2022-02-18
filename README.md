# 2022q1 Homework1 (lab0)
###### tags: `linux2022`
contributed by < [blue76815](https://github.com/blue76815/lab0-c) >

## 實驗環境

## [作業要求](https://hackmd.io/@sysprog/linux2022-lab0#-%E4%BD%9C%E6%A5%AD%E8%A6%81%E6%B1%82)

 - [x] 在 GitHub 上 fork [lab0-c](https://github.com/sysprog21/lab0-c)
- [ ] 依據上述指示著手修改 `queue.[ch]` 和連帶的檔案，測試後用 Git 管理各項修改，要能滿足 `$ make test` 自動評分系統的所有項目。
    - [x] 在提交程式變更前，務必詳閱 [如何寫好 Git Commit Message](https://blog.louie.lu/2017/03/21/%E5%A6%82%E4%BD%95%E5%AF%AB%E4%B8%80%E5%80%8B-git-commit-message/)
    - [ ] 詳閱「[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list)」並應用裡頭提及的手法，例如 pointer to pointer (指向某個指標的指標，或說 indirect pointer)，讓程式碼更精簡且有效
    - [ ] 用鏈結串列 ([linked list](https://en.wikipedia.org/wiki/Linked_list)) 來實作佇列 ([queue](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)))，請詳閱「[你所不知道的 C 語言: linked list 和非連續記憶體](https://hackmd.io/@sysprog/c-linked-list)」，以得知 Linux 核心原始程式碼風格的鏈結串列實作的考量，並研讀 [The Linux Kernel API - List Management Functions](https://www.kernel.org/doc/html/latest/core-api/kernel-api.html)。
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

## 作業實作
- [ ] `q_new`: 建立新的「空」佇列;
- [ ] `q_free`: 釋放佇列所佔用的記憶體;
- [ ] `q_insert_head`: 在佇列開頭 (head) 插入 (insert) 給定的新節點 (以 LIFO 準則);
- [ ] `q_insert_tail`: 在佇列尾端 (tail) 插入 (insert) 給定的新節點 (以 FIFO 準則);
- [ ] `q_remove_head`: 在佇列開頭 (head) 移去 (remove) 給定的節點;
- [ ] `q_release_element`: 釋放特定節點的記憶體空間;
- [ ] `q_size`: 計算目前佇列的節點總量;
- [ ] `q_delete_mid`: 移走佇列中間的節點，詳見 [LeetCode 2095. Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/)
- [ ] `q_delete_dup`: 在已經排序的狀況，移走佇列中具備重複內容的節點，詳見 [LeetCode 82. Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/)
- [ ] `q_swap`: 交換佇列中鄰近的節點，詳見 [LeetCode 24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)
- [ ] `q_reverse`: 以反向順序重新排列鏈結串列，該函式不該配置或釋放任何鏈結串列節點，換言之，它只能重排已經存在的節點;
- [ ] `q_sort`: 以==遞增順序==來排序鏈結串列的節點，可參閱 [Linked List Sort](https://npes87184.github.io/2015-09-12-linkedListSort/) 得知實作手法;