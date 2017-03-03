---
title: "Ukkonen's suffix tree construction - Part 5"
layout: post
comments: true
date: 2017-02-28
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

앞선 [Part 1][Part1], [Part 2][Part2], [Part 3][Part3], [Part 4][Part4]에서는 suffix tree의 기본 개념들, 고수준 Ukkonen 알고리즘, suffix link와 세 가지 구현 트릭, activePoint에 대한 몇 가지 세부 사항들, 그리고 예제 "abcabxabcd"에 대해서 6개의 phase를 거치며 suffix tree를 만들어 보았다.

여기서는 나머지 phase를 수행하여 tree를 완성해볼 것이다.

### Phase 7

Phase 7에서는 S(abcabxabcd$)의 7번째 문자 'a'를 읽는다.

- END를 7로 설정한다. 여태까지 6개의 leaf edge가 있기 때문에, 이 과정에서 extension 1~6이 완료된다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 1이 될 것이다. 즉, 1개의 extension이 수행될 것으로 기대된다).
- 만약 activeLength가 0이면(이전 phase에서 넘어온 activePoint는 (root, x, 0)이었다), activeEdge를 현재 추가할 문자로 설정한다. 이것은 **APCFALZ(ActivePoint Change For ActiveLength Zero)**이다. 이제 activePoint는 (root, 'a', 0)이 된다.
- activeNode에서 나가는 activeEdge가 있는지 확인한다. 만약 없다면, 새롭게 leaf edge를 만든다. 만약 있다면, walk-down한다. 우리의 예제에서는 outgoing edge 'a'가 activeNode(root)에 존재하므로, activeLength를 zero에서 1로 증가시키고(**APCFER3(ActivePoint Change For Extension Rule 3)**), phase를 종료한다.
- 이 시점에서 activePoint는 (root, a, 1)이며, remainingSuffixCount는 1이다.

Phase 7이 끝나는 시점에 remainingSuffixCount는 1이다(마지막 suffix 'a'는 tree에 암묵적으로 존재한다).

아래의 Figure 33은 phase 7 이후의 tree이다.

![Fig33]

### Phase 8

Phase 8에서는 S(abcabxabcd$)의 8번째 문자 b'를 읽는다.

- END를 8로 설정한다. 여태까지 6개의 leaf edge가 있기 때문에, 이 과정에서 extension 1~6이 완료된다. 이제 2개의 남은 suffix "ab"와 "b"를 추가해야 한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 2가 될 것이다. 즉, 2개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=2회) 루프를 돈다:
  - activeNode에서 나가는 activeEdge가 있는지 확인한다. 만약 없다면, 새롭게 leaf edge를 만든다. 만약 있다면, walk-down한다. 우리의 예제에서는, root에서 나가는 edge 'a'가 있으므로, walk-down을 해야 한다.
  - Trick 1 - skip/count를 이용하여 walk-down한다. 이번 phase 8에서는 activeLength(1) < edgeLength(2) 이므로, walkdown이 필요하지 않다.
  - 현재 추가해야 할 문자 b가 activePoint 다음에 존재하는지 확인한다. 만약 있다면, phase를 종료한다. 우리의 예제에서는 a 다음에 b가 있으므로, activeLength를 1에서 2로 1만큼 증가시키고, phase를 종료한다(Rule 3).
- 이 시점에서 activePoint는 (root, a, 2)이며, remainingSuffixCount는 2이다.

Phase 8이 끝나는 시점에 remainingSuffixCount는 2이다(마지막 suffix 'ab'와 'b'는 tree에 암묵적으로 존재한다).

아래의 Figure 34은 phase 8 이후의 tree이다.

![Fig34]


### Phase 9

Phase 9에서는 S(abcabxabcd$)의 9번째 문자 'c'를 읽는다.

- END를 9로 설정한다. 여태까지 6개의 leaf edge가 있기 때문에, 이 과정에서 extension 1~6이 완료된다. 이제 3개의 남은 suffix "abc"와 "bc", 그리고 "c"를 추가해야 한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 3가 될 것이다. 즉, 3개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=3회) 루프를 돈다:
  - activeNode에서 나가는 activeEdge가 있는지 확인한다. 만약 없다면, 새롭게 leaf edge를 만든다. 만약 있다면, walk-down한다. 우리의 예제에서는, root에서 나가는 edge 'a'가 있으므로, walk-down을 해야 한다.
  - Trick 1 - skip/count를 이용하여 walk-down한다. **이번 phase 9에서는 activeLength(2) >= edgeLength(2) 이므로, walkdown을 해야 한다.** Walkdown 시, activePoint는 (**APCFWD(ActivePoint Change For WalkDown)**)에 의해 (Node A, c, 0)로 변한다. 
  - 현재 추가해야 할 문자 c가 activePoint 다음에 존재하는지 확인한다. 만약 있다면, phase를 종료한다. 우리의 예제에서는 c로 시작하는 edge가 있으므로, activeLength를 0에서 1로 1만큼 증가시키고, phase를 종료한다(Rule 3).
- 이 시점에서 activePoint는 (Node A, c, 1)이며, remainingSuffixCount는 3이다.

Phase 9가 끝나는 시점에 remainingSuffixCount는 3이다(마지막 suffix 'abc'와 'bc', 'c'는 tree에 암묵적으로 존재한다).

아래의 Figure 35는 phase 9 이후의 tree이다.

![Fig35]

### Phase 10

Phase 10에서는 S(abcabxabcd$)의 10번째 문자 'd'를 읽는다.

- END를 10로 설정한다. 여태까지 6개의 leaf edge가 있기 때문에, 이 과정에서 extension 1~6이 완료된다. 이제 4개의 남은 suffix "abcd"와 "bcd", "cd", 그리고 "d"를 추가해야 한다.

![Fig36]

- remainingSuffixCount회(=4회) 루프를 돈다:

#### Phase 10 - Extension 7

- activeNode(Node A) 에서 나가는 activeEdge(c)가 있는지 확인한다. 아니라면, leaf edge를 만든다. 만약 있다면, walk-down한다. 우리의 예에서는 activeEdge(c)가 있기 때문에 walk-down을 시도한다.
- Extention 7에서는 activeLength(1) < edgeLength(8) 이므로 walk-down 하지 않아도 된다. 현재 넣고자 하는 문자('d')가 activePoint 다음에 나타나는지 확인한다. 만약 아니라면, Rule 2를 적용시킨다. 우리의 예에서는 activePoint에서 'd'로 시작하는 경로가 없기 때문에, 라벨 'd'를 갖는 새로운 leaf edge를 만든다. activePoint가 edge의 중간에서 끝나기 때문에 새로운 internal node를 activePoint 뒤에 생성한다(Rule 2)

![Fig37]

*새롭게 생성된 internal node C는 다음 extension에서 suffix link가 생길 것이다.(아래 Figure 38 참조)*

- suffix "abcd"가 명시적으로 tree에 추가되었으므로 remainingSuffixCount를 4에서 3으로 1만큼 감소시킨다.
- 이제 다음 extension 8을 위해서 activePoint를 바꿔야 한다. 현재의 activeNode는 internal node(Node A)이므로, 나가는 suffix link를 가진다. 이 suffix link를 따라 가면 Node B를 만나므로, 다음 activeNode는 Node B가 된다. 한편 activeEdge와 activeLength에는 변화가 없다(**APCFER2C2**). 따라서 새로운 activePoint는 (Node B, c, 1)이다.

#### Phase 10 - Extension 8

- 앞과 완전히 같은 논리로 문자 'd'를 추가한다. 앞의 extension 7에서, activePoint (Node A, c, 1)에 문자 'd'를 추가했고, 이번 extension 8에서는 같은 문자 'd'를 activepoint (Node B, c, 1)에 추가할 것이다. 따라서 같은 방법이 적용되며, 라벨 'd'를 갖는 새로운 leaf edge와 internal node가 생성될 것이다. 그리고 extension 7에서 생성된 internal node(C)는 새롭게 생긴 internal node(D)를 suffix link를 통해 가리키게 된다.

![Fig38]

*Extension 7에서 생성된 node C가 D로 향하는 suffix link를 가진 것을 확인하라. 이번에 생성된 node D는 다음 extension에서 자신의 suffix link를 가지게 될 것이다. 만약 다음 extension에서 새로운 node가 생기지 않는다면 어떻게 될까? 우리는 이러한 상황을 Phase 6([Part 4][Part4])에서 한번 봤고, Phase 10의 마지막 extension에서 한 번 더 보게 될 것이다. 계속해서 신경쓰도록 하라.*

- Suffix "bcd"가 tree에 명시적으로 더해졌으므로, remainingSuffixCount를 1만큼 감소시킨다(3에서 2로).
- 이제 activePoint를 extension 9를 위해 바꿔 줘야 한다. 현재의 activeNode는 internal node(Node B)이므로, suffix link를 갖는다. Suffix link를 따라가서 새로운 activeNode를 만날 수 있고(Root Node), activeEdge와 activeLength의 변화는 없다(**APCFER2C2**). 따라서 새로운 activePoint는 (root, c, 1)이다.

#### Phase 10 - Extension 9

- Suffix 'cd'를 추가한다. 앞과 완전히 같은 논리로 현재의 activePoint 뒤에 문자 'd'를 추가한다. Node D에서 E로 이어지는 suffix link가 생성된 것을 확인하라.

![Fig39]

- "cd"가 명시적으로 더해졌으므로, remainingSuffixCount를 1만큼 감소시킨다.
- 이제 activePoint를 extension 10을 위해 바꿔 줘야 한다. 현재의 activeNode는 root이고 activeLength는 1이므로, **APCFER2C1**에 의해 activeNode는 root로 바뀌지 않고, activeLength는 1만큼 감소될 것이며, activeEdge는 'd'가 될 것이다. 따라서 새로운 activePoint는 (root, d, 0)이다.

#### Phase 10 - Extension 10

- Extension 10에서는 suffix 'd'를 추가한다. 현재의 activePoint에 character 'd'를 추가하는데, 'd'로 시작하는 edge가 없으므로, 'd'로 라벨 붙여진 새로운 leaf edge를 추가한다(Rule 2). Extension 9에서 생성된 internal node E에서 root로 가는 suffix link가 생성된 것을 확인하라.

![Fig40]

*바로 다음의 extension에서 새로운 internal node가 생성되지 않는다면, 앞의 extension에서 생성된 internal node E에서 root로 가는 suffix link를 생성한다. 코드 구현에서는, 새로운 internal node가 생성되자마자 root로 향하는 suffix link를 생성한다. 만약 다음 extension에서 새로운 internal node가 생성된다면, suffix link를 새로 생성된 internal node로 향하게 하고, 만약 생성되지 않는다면, 그대로 root를 가리키게 하면 된다.*

- "d"가 명시적으로 tree에 더해졌으므로 remainingSuffixCount를 1만큼 감소시킨다(1에서 0으로). 더 추가할 suffix가 tree에 남아있지 않다는 의미이다. 현재 tree는 모든 suffix가 tree에 명시적으로 표현된 explicit tree임에 주목하라(왜? 문자 'd'를 이번 phase에 처음 만났기 때문이다!).
- Phase 11을 위한 activePoint는 (root, d, 0)이다.

Phase 10에서 다음과 같은 사실을 관찰할 수 있다:
- Suffix link로 연결된 internal node들 아래의 tree는 완전히 동일하다. 예를 들어, FIgure 40에서, A와 B 아래에는 완전히 동일한 tree가 달려 있다. 비슷하게, C, D, E 아래의 tree 또한 완전히 동일하다.
- 위의 사실에 의해, 임의의 extension에서 현재의 activeNode가 바로 이전의 activeNode에서 suffix link를 타고 와서 생긴 activeNode라면, 현재의 extension은 이전 extension과 완전히 동일한 논리로 진행된다 (Phase 10의 extension 7, 8, 9).
- Phase **i**의 extension **j**에서 새로운 internal node가 생성된다면, 해당 node는 바로 다음 extension **j+1**에서 suffix link를 갖게 된다. 
- 위의 사실에 의해, 모든 internal node는 다른 internal node 혹은 root로 향하는 suffix link를 갖는다. Root는 internal node가 아니기 때문에 suffix link를 갖지 않는다.

### Phase 11

Phase 11에서는 S(abcabxabcd$)의 10번째 문자 '$'를 읽는다.

- END를 11로 설정한다. 여태까지 총 10개의 leaf edge가 있기 때문에 이로 인해 extension 1, 2, ..., 10 까지 한번에 완료된다.

![Fig41]

- remainingSuffixCount를 1만큼 증가시킨다(0에서 1로). 즉, 이번 phase에서는 한 번의 extension이 수행될 것으로 기대된다.
- activeLength가 0이므로, activeEdge는 현재 추가될 문자 '$'로 바뀐다(**APCFALZ**).
- activeNode인 root에서 $로 시작하는 outgoing edge가 없으므로, 라벨 '$'를 갖는 새로운 leaf edge가 생성된다(Rule 2).
 
![Fig42]

- suffix '$'이 명시적으로 tree에 추가되었으므로, remainingSuffixCount를 1만큼 감소시킨다. 따라서 phase 11이 끝난 뒤에 명시적으로 추가될 suffix가 더 없다는 뜻이다. 현재 이 tree는 explicit tree임에 주목하라(왜? 문자 '$'를 이번 phase에 처음 만났기 때문이다).

이제 우리는 string 'abcabxabcd$'의 모든 suffix들을 suffix tree에 집어넣었다. tree에는 총 11개의 leaf end가 있고, root에서 leaf까지 가는 경로 상의 label을 합치면 하나의 suffix가 된다. 이제 해야할 단 한가지 일은 각각의 leaf에 suffix index 숫자를 부여하는 것이고, 그 숫자들은 문자열 S에서 suffix의 시작 위치가 될 것이다. 이것은 tree에서 DFS traversal로 구현 가능하다. DFS traversal 도중에는 label length를 저장해 두어야 하고, leaf end를 만났을 때 suffix index를 "stringSize - labelSize + 1"로 설정하면 된다. 완성된 suffix tree는 아래와 같다.

![Fig43]

*위의 그림에서 suffix indices의 position은 1부터 시작한다. 코드 구현에서는 zero-indexed일 것이다.*

**드디어 끝났다!!**

### Suffix tree를 표현하기 위한 자료 구조

어떻게 suffix tree를 표현할 수 있을까? Suffix tree에는 node와 edge, label, suffix link와 indices가 있다. 아래는 suffix tree를 구축할 때, 혹은 다른 응용에서 사용할 때 수행되어야 할 몇 가지 operation들이다.

- 어떤 edge가 주어졌을 때, path label의 길이를 구하라.
- 어떤 edge가 주어졌을 때, path label을 구하라.
- 어떤 node에서, 주어진 문자로 시작하는 outgoing edge가 있는지 확인하라.
- node에서 정해진 거리만큼 떨어진, edge 상의 문자는 무엇인가?
- suffix link를 통해 가리키고 있는 internal node를 얻어라.
- root에서 leaf까지 가는 경로의 suffix index를 구하라.
- 주어진 문자열이 suffix tree에 존재하는지 확인하라(substring, suffix, prefix).

생각하기로, 다양한 자료 구조가 이 요건을 충족시킬 수 있을 듯 하다.
다음 [Part 6][Part6]에서는 이를 만족시키는 자료 구조에 대해 논해볼 것이며, 실제로 코드 구현 예를 보일 것이다.

### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-5/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-1/
[2]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/
[Part1]: ../Ukkonen's-suffix-tree-construction-Part-1
[Part2]: ../Ukkonen's-suffix-tree-construction-Part-2
[Part3]: ../Ukkonen's-suffix-tree-construction-Part-3
[Part4]: ../Ukkonen's-suffix-tree-construction-Part-4
[Part5]: ../Ukkonen's-suffix-tree-construction-Part-5
[Part6]: ../Ukkonen's-suffix-tree-construction-Part-6

[Fig33]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig33-300x192.jpg
[Fig34]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig34-300x196.jpg
[Fig35]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig35-300x208.jpg
[Fig36]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig36-300x189.jpg
[Fig37]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig37-300x183.jpg
[Fig38]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig38-300x210.jpg
[Fig39]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig39-300x196.jpg
[Fig40]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig40-300x188.jpg
[Fig41]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig41-300x185.jpg
[Fig42]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig42-300x204.jpg
[Fig43]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig431-300x206.jpg