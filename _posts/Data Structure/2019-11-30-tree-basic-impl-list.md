---
layout: single
classes:
  - wide
title: 자료구조 - 리스트를 이용한 이진 트리 구현
comments: true
author_profile: true
---

### 기본 개념
[이 페이지](https://www.geeksforgeeks.org/binary-tree-set-1-introduction/)를 참고하여 정리하였습니다.

<pre>
       A
    &swarr;    &searr;
   B        C
 &swarr;  &searr;    &swarr;   &searr;
D     E  F     G
</pre>

위와 같은 트리가 있다고 할 때, 리스트를 이용하여 구현한다고 하면 하나의 Node는 자신의 왼쪽과 오른쪽에 어떤 노드가 오는지(어떤 자식 노드가 오는지) 알고 있어야 합니다.

만약 C를 이용하여 이를 구현한다 하면 Node 구조체를 아래와 같이 구성할 수 있을 것 같습니다.
~~~c
struct Node
{
  int data_;
  struct Node* left_;
  struct Node* right_;
}
~~~

구조체 안에는 자기 자신의 데이터를 담을 필드 하나와, 왼쪽 오른쪽 노드를 가리킬 포인터 하나씩 들고 있어야 합니다.

### 구현([전체 코드](https://github.com/SongKJ00/data-structure-study/blob/master/tree/tree_basic_with_list.cpp))
* Node 클래스

~~~cpp
class Node
{
public:
  char data_;   // 자신의 데이터
  Node* left_;  // 왼쪽 노드를 가리킬 포인터
  Node* right_; // 오른쪽 노드를 가리킬 포인터

public:
  /* 생성자 */
  Node(char data) :
    data_(data),
    left_(nullptr),
    right_(nullptr)
  {
  }
};
~~~

* Tree 클래스

~~~cpp
class Tree
{
public:
  Node* pRootNode_;

public:
  /* 루트 노드 setter */
  void SetRootNode(Node* rootNode)
  {
    pRootNode_ = rootNode;
  }

  /* 루트 노드 getter */
  Node* GetRootNode()
  {
    return pRootNode_;
  }

  /* 자신의 왼쪽 노드 setter */
  void SetLeft(Node* node, Node* parent)
  {
    parent->left_ = node;
  }

  /* 자신의 오른쪽 노드 setter */
  void SetRight(Node* node, Node* parent)
  {
    parent->right_ = node;
  }
};
~~~

* main 함수

~~~cpp
int main(void)
{
  /*       A
 *      <-  ->
 *    B       C
 *  <- ->   <- ->
 * D     E F     G
 * */

  Tree t;

  /* root node */
  Node* nA = new Node('A');
  t.SetRootNode(nA);

  /* level 2 */
  Node* nB = new Node('B');
  Node* nC = new Node('C');

  /* leaf nodes */
  Node* nD = new Node('D');
  Node* nE = new Node('E');
  Node* nF = new Node('F');
  Node* nG = new Node('G');

  /*    A
   *  <- ->
   * B     C
   */

  t.SetLeft(nB, nA);
  t.SetRight(nC, nA);

  /*  B-D-E subtree
   *    B
   *  <- ->
   * D     E
   */
  t.SetLeft(nD, nB);
  t.SetRight(nE, nB);

  /*  C-F-G subtree
   *    C
   *  <- ->
   * F     G
   */
  t.SetLeft(nF, nC);
  t.SetRight(nG, nC);

  return 0;
}
~~~