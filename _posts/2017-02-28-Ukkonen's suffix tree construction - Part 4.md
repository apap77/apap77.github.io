---
title: "Ukkonen's suffix tree construction - Part 4"
layout: post
comments: true
date: 2017-02-28
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

이전 글 [Part 1][Part1], [Part 2][Part2], [Part 3][Part3] 에서는 기본적인 suffix tree의 개념과, 고수준 Ukkonen 알고리즘, suffix link와 세 가지 구현 트릭, activePoint에 대한 몇가지 자세한 사항, 그리고 예제 string "abcabxabcd"에 대해 4번의 phase를 수행해 보았다. 

**activePoint**에 대해 알아보았으니, 앞선 4개의 phase를 다시 살펴보되, trick2, trick 3와 activePoint를 중점적으로 다루어 보자.
- activePoint는 처음에 (root, NULL, 0)으로 초기화한다. 즉, activeNode는 root이고, activeEdge는 NULL이며(편의상 여기서는 activeEdge의 값으로 문자를 주지만, 코드 구현에서는 문자의 index가 될 것이다), activeLength는 0이다.
- 전역 변수 END와 remainingSuffixCount는 0으로 초기화한다.

### Phase 1
Phase 1에서는 S의 첫 번째 문자 **'a'** 를 읽는다.
- END를 1로 설정한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 1이 될 것이다. 즉, 1개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=1회) 루프를 반복한다.
  - activeLength가 ZERO이면, activeEdge를 현재 문자로 바꾼다. 여기서는 activeEdge가 **'a'** 가 될 것이다. 이것은 **APCFALZ**이다.
  - activeNode(phase 1에서는 root)에서 activeEdge에 해당하는 outgoing edge가 존재하는지 확인한다. 아니라면, 새로 leaf edge를 만든다. 만약 존재한다면, walk-down한다. 우리의 예제에서는, Rule 2에 의해 새로운 leaf edge가 생성된다.
  - Extension이 수행되면, remainingSuffixCount를 1만큼 감소시킨다.
  - 이 시점에서 activePoint는 (root, a, 0)이다.

Phase 1의 종료 시점에서, remainingSuffixCount는 0이다(모든 suffix는 명시적으로 tree에 추가되었다). 아래는 그 결과 나타난 Tree이다.

![Fig20]

### Phase 2
Phase 2에서는 S의 두 번째 문자 **'b'** 를 읽는다.
- END를 2로 설정한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 1이 될 것이다. 즉, 1개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=1회) 루프를 반복한다.
  - activeLength가 ZERO이면, activeEdge를 현재 문자로 바꾼다. 여기서는 activeEdge가 **'b'** 가 될 것이다. 이것은 **APCFALZ**이다.
  - activeNode(phase 2에서는 root)에서 activeEdge에 해당하는 outgoing edge가 존재하는지 확인한다. 아니라면, 새로 leaf edge를 만든다. 만약 존재한다면, walk-down한다. 우리의 예제에서는, Rule 2에 의해 새로운 leaf edge가 생성된다.
  - Extension이 수행되면, remainingSuffixCount를 1만큼 감소시킨다.
  - 이 시점에서 activePoint는 (root, b, 0)이다.

Phase 2의 종료 시점에서, remainingSuffixCount는 0이다(모든 suffix는 명시적으로 tree에 추가되었다). 아래는 그 결과 나타난 Tree이다.

![Fig22]

### Phase 3
Phase 3에서는 S의 세 번째 문자 **'c'** 를 읽는다.
- END를 3으로 설정한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 1이 될 것이다. 즉, 1개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=1회) 루프를 반복한다.
  - activeLength가 ZERO이면, activeEdge를 현재 문자로 바꾼다. 여기서는 activeEdge가 **'c'** 가 될 것이다. 이것은 **APCFALZ**이다.
  - activeNode(phase 3에서는 root)에서 activeEdge에 해당하는 outgoing edge가 존재하는지 확인한다. 아니라면, 새로 leaf edge를 만든다. 만약 존재한다면, walk-down한다. 우리의 예제에서는, Rule 2에 의해 새로운 leaf edge가 생성된다.
  - Extension이 수행되면, remainingSuffixCount를 1만큼 감소시킨다.
  - 이 시점에서 activePoint는 (root, c, 0)이다.

Phase 3의 종료 시점에서, remainingSuffixCount는 0이다(모든 suffix는 명시적으로 tree에 추가되었다). 아래는 그 결과 나타난 Tree이다.

![Fig25]

### Phase 4
Phase 4에서는 S의 네 번째 문자 **'a'** 를 읽는다.
- END를 4으로 설정한다.
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 1이 될 것이다. 즉, 1개의 extension이 수행될 것으로 기대된다).
- remainingSuffixCount회(=1회) 루프를 반복한다.
  - activeLength가 ZERO이면, activeEdge를 현재 문자로 바꾼다. 여기서는 activeEdge가 **'a'** 가 될 것이다. 이것은 **APCFALZ**이다.
  - activeNode(phase 4에서는 root)에서 activeEdge에 해당하는 outgoing edge가 존재하는지 확인한다. 아니라면, 새로 leaf edge를 만든다. 만약 존재한다면, walk-down한다. 여기서는 Trick 1 - skip/count를 이용하여 walk-down 한다. 우리의 예에서는 activeNode(root)에서 나가는 edge 중 'a'로 시작하는 edge가 있다. 하지만 activeLength(0) < edgeLength(4) 이므로 walk-down은 필요하지 않다. activeLength를 0에서 1로 증가시키고(**APCFER3**), Phase를 종료한다(Rule 3).
  - 이 시점에서 activePoint는 (root, a, 1)이고, remainingSuffixCount는 1이 된다.

Phase 4의 종료 시점에서, remainingSuffixCount는 1이다(하나의 suffix, 마지막 'a'는 명시적으로 tree에 추가되지 않았지만, 암묵적으로 tree 안에 나타나 있다). 아래는 그 결과 나타난 Tree이다.

![Fig28]

여기까지가 [Part 3][Part3]에서 다루었던 Phase 1~4 에 대한 내용이었다. Tree를 계속하여 만들어 보자.
***

### Phase 5
Phase 5에서는 S의 다섯 번째 문자 **'b'** 를 읽는다.
- END를 5으로 설정한다(이것으로 extension 1, 2, 3은 완료된다).
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 2가 될 것이다. 즉, 2개의 extension(4, 5)가 수행될 것으로 기대된다). Extension 4는 suffix "ab"를, extension 5는 suffix "b"를 추가할 것이다.
- remainingSuffixCount회(=2회) 루프를 반복한다.
  - activeNode(phase 5에서는 root)에서 activeEdge에 해당하는 outgoing edge가 존재하는지 확인한다. 아니라면, 새로 leaf edge를 만든다. 만약 존재한다면, walk-down한다. 여기서는 Trick 1 - skip/count를 이용하여 walk-down 한다. 우리의 예에서는 activeNode(root)에서 나가는 edge 중 'a'로 시작하는 edge가 있다. 하지만 activeLength(1) < edgeLength(4) 이므로 walk-down은 필요하지 않다. activeLength를 1에서 2로 증가시키고(**APCFER3**), Phase를 종료한다(Rule 3).
  - 이 시점에서 activePoint는 (root, a, 2)이고, remainingSuffixCount는 2가 된다.

Phase 5의 종료 시점에서, remainingSuffixCount는 2이다(suffix "ab"와 "b"는 명시적으로 추가되지 않았지만, 암묵적으로 tree 내에 존재한다). 아래는 그 결과 나타난 Tree이다.

![Fig29]

### Phase 6
Phase 6에서는 S의 여섯 번째 문자 **'x'** 를 읽는다.
- END를 6으로 설정한다(이것으로 extension 1, 2, 3은 완료된다).
- remainingSuffixCount를 1만큼 증가시킨다(remainingSuffixCount는 3가 될 것이다. 즉, 3개의 extension(4, 5, 6)이 수행될 것으로 기대된다). Extension 4는 suffix "abx"를, extension 5는 suffix "bx"를, extension 6는 suffix "x"를 추가할 것이다.
- remainingSuffixCount회(=3회) 루프를 반복한다.
  - Extension 4 에서 activePoint는 (root, a, 2)이며, 'a'로 시작하는 edge 상의 두 번째 문자 'b'를 가리킨다.
  - Extension 4에서 현재 넣어야 하는 문자 'x'는 activePoint 다음의 문자와 일치하지 않는다. 따라서 이것은 extension Rule 2에 해당하는 상황이다. 따라서 여기에 **x**로 라벨 붙여진 leaf edge가 추가된다. 그리고 경로 탐색은 edge의 중간에서 끝나므로, activePoint의 끝에 새로운 internal node가 생성된다.
  - 명시적으로 suffix "abx"를 추가했으므로, remainingSuffixCount를 1만큼 감소시킨다(3에서 2로).

아래는 현재까지 만들어진 tree이다.

![Fig30]

Rule 2가 적용되고 난 이후 activePoint는 바뀔 것이다. 세 가지의 다른 케이스(**APCFER3, APCFWD, 그리고 APCFALZ**)는 이미 [Part 3][Part3]에서 다루었다.

**activePoint change for extension rule 2 (APCFER2):**

**Case 1 (APCFER2C1):** *만약 activeNode가 root이고, activeLength가 0보다 크다면* activeLength를 1만큼 감소시키고, activeEdge는 "S[i - remainingSuffixCount + 1]"로 설정된다(**i**는 현재의 phase number). 왜 activePoint가 이렇게 변했는지 알겠는가? Phase 6에서 "abx"를 추가했던 위의 extension으로 돌아가 보자. activeLength는 2이며 activeEdge는 'a'이다. 다음 extension에서, 우리는 suffix "bx"를 더해야 한다. 즉, 다음 extension에서 path label은 'b'로 시작해야 한다. 따라서 S의 5번째 문자인 'b'가 다음 extension에서의 activeEdge가 되어야 하며, 이 때 b의 index는 "i - remainingSuffixCount + 1" (6 - 2 + 1 = 5)가 된다. activeLength는 1만큼 감소하는데, 매 extension마다 activePoint는 root를 향하여 1 문자씩 가까워지기 때문이다.

만약 *activeNode가 root이고, activeLength가 0이면* 어떻게 될까? 이 케이스는 이미 **APCFALZ**에서 다룬 케이스이다.

**Case 2 (APCFER2C2): *만약 activeNode가 root가 아니면,* 현재 activeNode의 suffix link를 따라 간다. suffix link가 가리키는 새로운 node는 다음 extension의 activeNode가 될 것이다. activeLength와 activeEdge에 변화는 없다. 왜 activePoint가 이렇게 변했는지 알겠는가? 왜냐하면, *만약 두 node가 suffix link로 연결되어 있다면, 그 두 node로부터 시작되어 아래로 이어지는, 같은 문자로 시작하는 모든 경로의 쌍들은 정확히 일치할 것이며, activePoint가 이전과 이후, 두 경로에서 동일한 위치에 존재하기 때문에 activeEdge와 activeLength의 변화는 없다.* ([Part 2][Part2]의 Figure 18을 참조)

![Fig18]

예를 들어 phase **i**의 extension **j**에 대해서, tree에 suffix 'xAabcdedg'가 추가되었다고 생각하자. 그 시점에, activePoint는 (Node-V, a, 7)이었다고 하자('g'를 나타냄). 따라서 그 다음 extension **j+1**에 대해서 우리는 suffix 'Aabcdedg'를 추가해야 하고, Figure 18에 나타난 오른쪽 path를 따라서 탐색을 해야 한다. 이것은 activeNode **v**에서 suffix link를 타고 이동함으로써 수행될 수 있다. Suffix link는 이전 activeNode의 아랫 부분의 경로와 정확히 동일한 경로를 따라 이동할 수 있는 어떤 node **s(v)** 로 우리를 이동시킨다. 앞서 말했듯이, 매 extension마다 activePoint는 root를 향하여 1씩 가까워진다. 이 경우 이 변화는 node **s(v)** 위쪽에서 일어나게 되고, node **s(v)** 아랫 부분에는 아무런 변화가 없게 된다. 따라서 activeNode가 현재 extension에서 root가 아니라면, 다음 extension에서는 activeNode만 바뀌게 되는 것이다(activeEdge나 activeLength의 변화 없이).

- Extension 4가 수행될 때 activePoint는 (root, a, 2)이며, **APCFER2C1**에 의해서 extension 5를 위한 activePoint는 (root, b, 1)이 될 것이다.
- 다음으로 더해질 suffix는 "bx"이다. 이 때, remainingSuffixCount는 2이다.
- 현재 넣어야 하는 문자 'x'는 activePoint 다음 문자와 일치하지 않는다. 따라서 이것은 extension Rule 2의 상황이다. 따라서 leaf edge가 만들어지며, 라벨은 **x**이다. 또한 경로 탐색이 edge의 중간에서 끝났기 때문에, 새로운 internal node가 activePoint 뒤에 생성된다. Suffix link가 형성되는데, 앞선 internal node(extension 4에서 생성된)에서 시작하여 현재의 internal node를 향하는 suffix link가 생성된다.
- 명시적으로 suffix "bx"가 tree에 더해졌으므로, remainingSuffixCount를 1만큼 감소시킨다(2에서 1로).

![Fig31]

- Extension 5가 수행될 때 activePoint는 (root, b, 1)이며, **APCFER2C1**에 의해서 extension 6를 위한 activePoint는 (root, x, 0)이 될 것이다. 
- 새롭게 추가되어야 할 suffix는 "x"이다. 이 때, remainingSuffixCount는 1이다. 
- Extension 6에서는, root에서 나오는 어떤 edge와도 일치하지 않을 것이므로, **x**로 라벨 붙여진 새로운 edge가 root node로부터 나오도록 생성될 것이다. 또한 앞선 internal node(extension 5에서 생성된)에서 나와 root를 가리키는 suffix link를 생성한다.
- "x"가 명시적으로 tree에 더해졌으므로, remainingSuffixCount를 1만큼 감소시킨다(1에서 0으로).

![Fig32]

이렇게 phase 6이 완료되었다.

Phase 6은 6개의 모든 extension을 완료했음에 주목하라(왜냐하면, 추가되는 문자 **x**는 여태껏 한 번도 나타나지 않은 문자이기 때문에, Rule 3이 적용될 수 없기 때문이다). 따라서 phase 6의 결과 생성된 suffix tree는 현재까지 만든 문자열 "abcabx"에 대해서 tree suffix tree(explicit suffix tree)이다. 

위의 tree를 만드는 과정에서 아래와 같은 사실을 발견하였다:
- Extension **i**에서 새롭게 만들어진 internal node는 extension **i+1**이 종료되는 시점에 다른 internal node 혹은 root(만약 extension i+1가 사용하는 activeNode가 root인 경우)를 suffix link를 통해 가리키고 있다(모든 internal node는 **반드시** 다른 internal node나 root를 가리키는 suffix link를 가져야 한다).
- 다음 suffix를 path-label로 갖는 경로의 끝을 찾을 때, suffix link는 지름길을 제공한다.
- Extension과 phases 간에 적절하게 activePoint를 관리한다면 root로부터의 불필요한 walkdown을 피할 수 있다.

[Part 5][Part5]에서는 나머지 phases(7 에서 11) 대해 마저 연습을 해 보고 완성된 suffix tree를 얻을 것이며, [Part 6][Part6]에서는 코드 구현에 대해 알아볼 것이다.

### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-4/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-4/
[2]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/
[Part1]: ../Ukkonen's-suffix-tree-construction-Part-1
[Part2]: ../Ukkonen's-suffix-tree-construction-Part-2
[Part3]: ../Ukkonen's-suffix-tree-construction-Part-3
[Part5]: ../Ukkonen's-suffix-tree-construction-Part-5
[Part6]: ../Ukkonen's-suffix-tree-construction-Part-6

[Fig18]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig18-297x300.jpg
[Fig20]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig20-300x137.jpg
[Fig22]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig22-300x161.jpg
[Fig25]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig25-300x159.jpg
[Fig28]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig28.jpg
[Fig29]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig29.jpg
[Fig30]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig30-300x211.jpg
[Fig31]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig31-300x223.jpg
[Fig32]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig32-300x203.jpg