# 第四章 树


<!--more-->

# 二叉查找树

对于树中的每个节点X，它的左子树中所有关键字值小于X的关键值，它的右子树中所有的关键值大于X的关键值

## 数据结构

```c
#ifndef _Tree_H
typedef int ElementType;
struct TreeNode;
typedef struct TreeNode *Postion;
typedef struct TreeNode *SearchTree;

SearchTree MakeEmpty(SearchTree T);
Postion Find(ElementType e, SearchTree T);
Postion FindMin(SearchTree T);
Postion FindMax(SearchTree T);
SearchTree Insert(ElementType e, SearchTree T);
ElementType Retrieve(Postion p);
#endif

typedef struct TreeNode {
  ElementType Element;
  SearchTree Left;
  SearchTree Right;
} TreeNode;

```

## 具体实现

### 插入

```c
SearchTree Insert(ElementType e, SearchTree T) {
  if (T == NULL) {
    T = (TreeNode *)malloc(sizeof(TreeNode));
    if (T == NULL) {
      printf("out of space !");
    } else {
      T->Element = e;
      T->Left = T->Right = NULL;
    }
  } else if (e < T->Element) {
    T->Left = Insert(e, T->Left);
  } else if (e > T->Element) {
    T->Right = Insert(e, T->Right);
  }
  return T;
}
```

### 最大值

````c
Postion FindMax(SearchTree T) {
  if (T == NULL) {
    return NULL;
  } else if (T->Right == NULL) {
    return T;
  } else {
    return FindMax(T->Right);
  }
}
````

### 最小值

```c
Postion FindMin(SearchTree T) {
  if (T == NULL) {
    return NULL;
  } else if (T->Left == NULL) {
    return T;
  } else {
    return FindMin(T->Left);
  }
}
```

### 查找某个节点

````c
Postion Find(ElementType e, SearchTree T) {
  if (T == NULL)
    return NULL;
  if (T->Element < e) {
    return Find(e, T->Right);
  } else if (T->Element > e) {
    return Find(e, T->Left);
  } else {
    return T;
  }
}
````

### 初始化

```c
SearchTree MakeEmpty(SearchTree T) {
  if (T != NULL) {
    MakeEmpty(T->Left);
    MakeEmpty(T->Right);
    free(T);
  }
  return NULL;
}
```

### 测试

````c
int main() {
  TreeNode *root = Insert(0, NULL);
  Insert(2, root);
  Insert(3, root);
  Insert(-1, root);
  Insert(-2, root);
  Insert(-3, root);
  TreeNode *min = FindMin(root);
  printf("min:%d\n", min->Element);
  TreeNode *max = FindMax(root);
  printf("max:%d\n", max->Element);
  TreeNode *some = Find(2, root);
  printf("some:%d\n", some->Element);
  root = MakeEmpty(root);
  printf("some:%d\n", root == NULL);
}
````

### 删除元素

````c
SearchTree Delete(ElementType e, SearchTree T) {
  if (T == NULL) {
    printf("not found target\n");
  }
  SearchTree tmpCell;
  if (e < T->Element) {
    T->Left = Delete(e, T->Left);
  } else if (e > T->Element) {
    T->Right = Delete(e, T->Right);
  } else if (T->Right && T->Left) {
    tmpCell = FindMin(T->Right);
    T->Element = tmpCell->Element;
    T->Right = Delete(T->Element, T->Right);
  } else {
    tmpCell = T;
    if (T->Left == NULL) {
      T = T->Right;
    } else if (T->Right == NULL) {
      T = T->Left;
    }
    free(tmpCell);
  }
  return T;
}
````

### 测试删除

````c
int main() {
  TreeNode *root = Insert(0, NULL);
  Insert(2, root);
  Insert(3, root);
  Insert(-1, root);
  Insert(-2, root);
  Insert(-3, root);
  printf("max:%d\n", FindMax(root)->Element);
  Delete(3, root);
  printf("max:%d\n", FindMax(root)->Element);
}
````




