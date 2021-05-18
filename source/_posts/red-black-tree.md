---
title: 红黑树
tags: [tree]
abbrlink: f89cb603
date: 2021-03-29 19:19:41
---

```java
左旋
void leftRotate(Tree t, TreeNode x){
  TreeNode y = x.right; // 取得x节点的右儿子赋值给y

  x.right = y.left;     // 将y的左儿子赋值给x的右儿子
  y.left.parent = x;    // 将y的左儿子的父节点变更成x

  y.parent = x.parent;   // 将x的父节点赋值给y的父节点
  if(x.parent == null){
    t.root = y;
  }else if(x == x.parent.left){
    x.parent.left = y;
  }else{
    x.parent.right = y;
  }
  y.left = x;
  x.parent = y;
}
```

```java
插入

void RBInsert(Tree t, TreeNode z){
  TreeNode y = null;
  TreeNode x = t.root;
  while(x != null){
    y = x;
    if(z.value < x.value){
      x = x.left;
    }else{
      x = x.right;
    }
  }

  z.parent = y;

  if(y == null){
    t.root = z;
  }else if(z.value < y.value){
    y.left = z;
  }else{
    y.right = z;
  }

  z.left = null;
  z.right = null;
  z.color = RED;
  RBInsertFixUp(t,z);
}

```

```java
红黑树插入后调整

void RBInsertFixUp(Tree t,Node z){
  while(z.parent.color == RED){ 
    if(z.parent == z.parent.parent.left){
      TreeNode y = z.parent.parent.right;
      if(y.color == RED){
        z.parent.color = BLACK;
        y.color = BLACK;
        z.parent.parent.color = RED;
        z = z.parent.parent;
      }else if(z == z.parent.right){
        z = z.parent;
        leftRotate(t,z);
        z.parent.color = BLACK;
        z.parent.parent.color = RED;
        rightRotate(t,z.parent.parent);
      }
    }else{
      // 同上，调转left和right
    }
  }
  t.root.color = BLACK;

}
```
