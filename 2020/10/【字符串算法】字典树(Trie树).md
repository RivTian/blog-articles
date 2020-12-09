---
title: "【字符串算法】字典树(Trie树)"
date: 2020-10-08T15:55:42+08:00
draft: false
tags:
  - 字符串
  - Trie
topics:
  - 
---

### 字典树

　　字典树，又称单词查找树，`Trie`树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来节约存储空间，最大限度地减少无谓的字符串比较，查询效率比哈希表高。
　　字典树与字典很相似,当你要查一个单词是不是在字典树中,首先看单词的第一个字母是不是在字典的第一层,如果不在,说明字典树里没有该单词,如果在就在该字母的孩子节点里找是不是有单词的第二个字母,没有说明没有该单词,有的话用同样的方法继续查找.字典树不仅可以用来储存字母,也可以储存数字等其它数据。

**Trie的数据结构定义：**

```cpp
#define MAX 26
typedef struct Trie {
    Trie* next[MAX];
    int v;  //根据需要变化
};

Trie* root;
```

　　next是表示每层有多少种类的数，如果只是小写字母，则26即可，若改为大小写字母，则是52，若再加上数字，则是62了，这里根据题意来确定。 v可以表示一个字典树到此有多少相同前缀的数目，这里根据需要应当学会自由变化。

　　Trie的查找（最主要的操作）：

　　(1) 每次从根结点开始一次搜索；
　　(2) 取得要查找关键词的第一个字母，并根据该字母选择对应的子树并转到该子树继续进行检索； 　　

​	   (3) 在相应的子树上，取得要查找关键词的第二个字母,并进一步选择对应的子树进行检索。 　　 　   (4) 迭代过程……
　   (5) 在某个结点处，关键词的所有字母已被取出，则读取附在该结点上的信息，即完成查找。

　　这里给出生成字典树和查找的模版：

**生成字典树：**

```c++
void createTrie(char* str) {
    int len = strlen(str);
    Trie *p = root, *q;
    for (int i = 0; i < len; ++i) {
        int id = str[i] - '0';
        if (p->next[id] == NULL) {
            q = (Trie*)malloc(sizeof(Trie));
            q->v = 1;  //初始v==1
            for (int j = 0; j < MAX; ++j)
                q->next[j] = NULL;
            p->next[id] = q;
            p = p->next[id];
        } else {
            p->next[id]->v++;
            p = p->next[id];
        }
    }
    p->v = -1;  //若为结尾，则将v改成-1表示
}
```

**查找:**

```c++
int findTrie(char* str) {
    int len = strlen(str);
    Trie* p = root;
    for (int i = 0; i < len; ++i) {
        int id = str[i] - '0';
        p = p->next[id];
        if (p == NULL)  //若为空集，表示不存以此为前缀的串
            return 0;
        if (p->v == -1)  //字符集中已有串是此串的前缀
            return -1;
    }
    return -1;  //此串是字符集中某串的前缀
}
```



#### 例题

- [hdu 1251 统计难题](http://acm.hdu.edu.cn/showproblem.php?pid=1251)

　　题意：在给出的字符串中找出由给出的字符串中出现过的两个串拼成的字符串。
　　字典树的模板题，先建字典数，然后再查询每个给定的单词。。

**代码如下:**

```c++
#include <string.h>
#include <iostream>
using namespace std;

const int sonsum = 26, base = 'a';
char s1[12], ss[12];

struct Trie {
    int num;
    bool flag;
    struct Trie* son[sonsum];
    Trie() {
        num = 1;
        flag = false;
        memset(son, NULL, sizeof(son));
    }
};

Trie* NewTrie() {
    Trie* temp = new Trie;
    return temp;
}

void Inset(Trie* root, char* s) {
    Trie* temp = root;
    while (*s) {
        if (temp->son[*s - base] == NULL) {
            temp->son[*s - base] = NewTrie();
        } else
            temp->son[*s - base]->num++;
        temp = temp->son[*s - base];
        s++;
    }
    temp->flag = true;
}

int search(Trie* root, char* s) {
    Trie* temp = root;
    while (*s) {
        if (temp->son[*s - base] == NULL)
            return 0;
        temp = temp->son[*s - base];
        s++;
    }
    return temp->num;
}

int main() {
    Trie* root = NewTrie();
    root->num = 0;
    // while(cin.get(s1,12))
    while (gets(s1) && strcmp(s1, "") != 0) {
        // if(strcmp(s1," ")==0)
        // break;
        Inset(root, s1);
    }
    while (cin >> ss) {
        int ans = search(root, ss);
        cout << ans << endl;
    }

    return 0;
}
```

- [poj 2001 Shortest Prefixes](http://poj.org/problem?id=2001)

　　题意：找出能唯一标示一个字符串的最短前缀，如果找不出，就输出该字符串。
　　用字典树即可
　　
**代码如下：**

```c++
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <iostream>
using namespace std;

char list[1005][25];

struct node {
    int count;
    node* childs[26];
    node() {
        count = 0;
        int i;
        for (i = 0; i < 26; i++)
            childs[i] = NULL;
    }
};

node* root = new node;
node *current, *newnode;

void insert(char* str) {
    int i, m;
    current = root;
    for (i = 0; i < strlen(str); i++) {
        m = str[i] - 'a';
        if (current->childs[m] != NULL) {
            current = current->childs[m];
            ++(current->count);
        } else {
            newnode = new node;
            ++(newnode->count);
            current->childs[m] = newnode;
            current = newnode;
        }
    }
}

void search(char* str) {
    int i, m;
    char ans[25];
    current = root;
    for (i = 0; i < strlen(str); i++) {
        m = str[i] - 'a';
        current = current->childs[m];
        ans[i] = str[i];
        ans[i + 1] = '\0';
        if (current->count == 1)  //可以唯一标示该字符串的前缀
        {
            printf("%s %s\n", str, ans);
            return;
        }
    }
    printf("%s %s\n", str, ans);  // 否则输出该字符串
}

int main() {
    int i, t = 0;
    while (scanf("%s", list[t]) != EOF) {
        insert(list[t]);
        t++;
    }
    for (i = 0; i < t; i++)
        search(list[i]);
    return 0;
}
```

- [hdu 4825 Xor Sum](http://acm.hdu.edu.cn/showproblem.php?pid=4825)

　　题意：给你一些数字，再询问Q个问题，每个问题给一个数字，使这个数字和之前给出的数字的异或和最大。
　　构造字典树，高位在前，低位在后，然后顺着字典树根向深处递归查询


**代码如下：**

```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <functional>
#include <iostream>
#include <map>
#include <queue>
#include <set>
#include <string>
#include <vector>

using namespace std;
typedef long long LL;
typedef pair<LL, int> PLI;

const int MX = 2e5 + 5;
const int INF = 0x3f3f3f3f;

struct Node {
    Node* Next[2];
    Node() { Next[0] = Next[1] = NULL; }
};

void trie_add(Node* root, int S) {
    Node* p = root;
    for (int i = 31; i >= 0; i--) {
        int id = ((S & (1 << i)) != 0);
        if (p->Next[id] == NULL) {
            p->Next[id] = new Node();
        }
        p = p->Next[id];
    }
}

int trie_query(Node* root, int S) {
    Node* p = root;
    int ans = 0;
    for (int i = 31; i >= 0; i--) {
        int id = ((S & (1 << i)) != 0);
        if (p->Next[id ^ 1] != NULL) {
            ans |= (id ^ 1) << i;
            p = p->Next[id ^ 1];
        } else {
            ans |= id << i;
            p = p->Next[id];
        }
    }
    return ans;
}

int main() {
    // freopen("input.txt", "r", stdin);
    int T, n, Q, t, ansk = 0;
    scanf("%d", &T);
    while (T--) {
        scanf("%d%d", &n, &Q);
        Node* root = new Node();

        for (int i = 1; i <= n; i++) {
            scanf("%d", &t);
            trie_add(root, t);
        }

        printf("Case #%d:\n", ++ansk);
        while (Q--) {
            scanf("%d", &t);
            printf("%d\n", trie_query(root, t));
        }
    }
    return 0;
}
```