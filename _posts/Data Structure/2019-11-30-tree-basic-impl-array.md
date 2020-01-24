---
layout: single
classes:
  - wide
title: 자료구조 - 배열을 이용한 이진 트리 구현
comments: true
author_profile: true
---

### 기본 개념
[이 페이지](https://www.geeksforgeeks.org/binary-tree-array-implementation/)를 참고하여 정리하였습니다.

<pre>
       A
    &swarr;    &searr;
   B        C
 &swarr;  &searr;    &swarr;   &searr;
D     E  F     G
</pre>

위와 같은 트리가 있다고 할 때, 배열을 이용하여 구현하면 아래와 같이 각각 노드에 인덱스를 부여할 수 있습니다.

<pre>
       A(0)
    &swarr;    &searr;
   B(1)      C(2)
 &swarr;  &searr;      &swarr;   &searr;
D(3) E(4) F(5)   G(6)
</pre>

B 인덱스(1) = A 인덱스(0) * 2 + 1<br>
C 인덱스(2) = A 인덱스(0) * 2 + 2<br>

D 인덱스(3) = B 인덱스(1) * 2 + 1<br>
E 인덱스(4) = B 인덱스(1) * 2 + 2<br>

... 같이 계산이 가능하므로 아래와 같은 모든 노드의 인덱스를 구할 수 있습니다.

<pre>
루트 노드 인덱스 = 0
왼쪽 자식 노드 인덱스 = 부모 노드 인덱스 * 2 + 1
오른쪽 자식 노드 인덱스 = 부모 노드 인덱스 * 2 + 2
</pre>

트리의 노드가 총 n개라고 하면 노드의 인덱스의 범위는 0 ~ n-1입니다.
<br><br><br>

### 구현([전체 코드](https://github.com/SongKJ00/data-structure-study/blob/master/tree/tree_basic_with_array.cpp))
* Tree 클래스

~~~cpp
class Tree
{
public:
  std::vector<char> nodes_; // 노드들을 담을 공간(벡터 이용)

public:
  /* Constructor */
  /* 루트 노드와 트리의 전체 사이즈를 정합니다. */
  Tree(int node_size, char root_data)
  {
    nodes_.resize(node_size);
    nodes_[0] = root_data;
  }

  /* 부모 인덱스를 받아 왼쪽 자식 노드를 설정합니다. */
  void SetLeft(char data, int parent_idx)
  {
    nodes_[parent_idx*2+1] = data;
  }

  /* 부모 인덱스를 받아 오른쪽 자식 노드를 설정합니다. */
  void SetRight(char data, int parent_idx)
  {
    nodes_[parent_idx*2+2] = data;
  }

  /* 트리의 모든 노드 출력 */
  void PrintTree()
  {
    for(auto& node: nodes_)
    {
      std::cout << node << " ";
    }
    std::cout << std::endl;
  }
};
~~~

* main 함수

~~~cpp
int main(void)
{
  /*       A(0)
  *       <-  ->
  *    B(1)      C(2)
  *  <- ->      <- ->
  * D(3)  E(4) F(5)  G(6)
  * */

  Tree t(7, 'A');
  t.SetLeft('B', 0);
  t.SetRight('C', 0);

  t.SetLeft('D', 1);
  t.SetRight('E', 1);

  t.SetLeft('F', 2);
  t.SetRight('G', 2);

  t.PrintTree();

  return 0;
}
~~~