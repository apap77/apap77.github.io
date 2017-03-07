---
title: "Ukkonen's suffix tree construction - Part 3"
layout: post
comments: true
date: 2017-02-28
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

*Suffix tree의 기본적인 개념과, 고수준 Ukkonen 알고리즘, suffix link, 그리고 세 가지의 구현 트릭에 대해 다룬 [Part 1][Part1]과 [Part 2][Part2]에 이어지는 글이다.*

String **S = abcabxabcd**를 예로 들어서 단계적으로 suffix tree를 만들어 볼 것이다. 먼저 **$** 문자를 추가하여 **S = abcabxabcd$** 을 만들고 시작한다.

길이 m인 string **S** 에 대해서 suffix tree를 만들 때:

- 1부터 m까지 총 **m**개의 phase가 존재한다. 우리의 예에서는 **m = 11**이므로, 총 11개의 phase가 존재할 것이다.
- 첫 번째 phase에서는 문자 'a'를 tree에 추가할 것이고, 두 번째 phase에서는 문자 'b'를 tree에 추가할 것이고, ..., **m**번째 phase에서는 **m**번째 문자를 tree에 추가할 것이다. (이러한 특성 때문에 Ukkonen 알고리즘은 online 알고리즘이다.)
- Phase **i**는 **최대 i개의 extension**을 가진다(1부터 i까지). 만약 현재 집어넣어야 할 문자가 이전 phase들에서 한번도 만난 적 없는 문자라면, 모든 **i**개의 extension이 수행될 것이다(즉, Rule 3가 적용되지 않는 phase가 될 것이다). 반면 현재 집어넣어야 할 문자가 이미 한 번 이상 나타났던 문자라면, phase **i**는 **i**번의 extension을 다 수행하지는 않고 조기에 완료될 것이다(Rule 3이 적용되자마자). 
- 총 3가지 extension rule이 있고, 임의의 phase의 각각의 extension **j**(1부터 i까지)는 이 3가지의 extension rule 중 하나를 따른다.
- Rule 1은 존재하는 leaf edge에 새로운 문자를 추가한다.
- Rule 2는 새로운 leaf edge를 만든다(만약 path label이 edge 중간에서 끝난다면, internal node가 생성된다).
- Rule 3은 현재 phase를 종료한다.

![Fig20]

- Phase 1에서는 string의 첫 번째 문자를 읽고, 한 번의 extension이 수행될 것이다 **(그림 상에서는 편의상 edge label에 문자를 적어 두었지만, 실제 구현에서는 [Part 2][Part2]에서 다루었던 edge-label compression을 이용하여 (start, end) indices만 저장된다)**. Extension 1은 suffix "a"를 tree에 집어넣을 것이다. Root로부터 시작하여 label 'a'를 갖는 경로를 탐색한다. Root로부터 나오는 경로가 하나도 없기 때문에, 새로운 leaf edge를 만든다(Rule 2). Phase 1는 extension 1이 끝남으로써 종료된다. 어떤 string에 대해서도 phase 1은 Rule 2가 적용되는 하나의 extension만을 갖는다.

***

- Phase 2는 두 번째 문자를 읽고, 적어도 1번, 최대 2번의 extension이 수행된다. 우리의 예에서, phase 2에서는 두 번째 문자 'b'를 읽고 tree에 집어넣을 것이다. 추가될 suffix들은 "ab"와 "b"이다.

![Fig21]

- Extension 1에서는 suffix "ab"가 tree에 넣어질 것이다. Label "a"를 갖는 경로가 leaf edge에서 끝나므로, 이 edge의 끝에 'b'를 추가한다(Rule 1). Extension 1은 그냥 첫 번째 edge에 대해 end index를 1만큼 증가시킴으로써(1에서 2로) 끝난다.

![Fig22]

- Extension 2에서는 suffix "b"를 tree에 넣는다. Root에서 label이 b로 시작하는 edge가 없으므로, 새로운 leaf edge를 만든다(Rule 2).

- Phase 2는 extension 2가 끝남으로써 종료된다. Phase 2에서는 2번의 extension이 수행되었다. 첫 번째 extension에서는 Rule 1이, 두 번째 extension에서는 Rule 2가 적용되었다.

***

- Phase 3에서는 세 번째 문자를 읽을 것이며, 적어도 1번, 최대 3번의 extension이 수행될 것이다. 우리의 예에서는 세 번째 문자 'c'를 tree에 넣을 것이며, 추가될 suffix들은 "abc", "bc", "c"의 3개이다.

![Fig23]

- Extension 1에서는 "abc"를 tree에 넣는다. Label "ab"를 갖는 경로가 leaf edge에서 끝나므로, 이 edge의 끝에 'c' 를 추가한다(Rule 1). Extension 1은 그냥 이 edge에 대해 end index를 1만큼 증가시킴으로써(2에서 3으로) 끝난다.

![Fig24]

- Extension 2에서는 suffix "bc"를 tree에 넣는다. Label "b"를 갖는 경로가 leaf edge에서 끝나므로, 이 edge의 끝에 'c'를 추가한다. Extension 2는 그냥 이 edge에 대해 end index를 1만큼 증가시킴으로써(2에서 3으로) 끝난다.

![Fig25]

- Extension 3에서는 suffix "c"를 tree에 넣는다. Root에서 label 'c'를 갖는 path가 없으므로, 새로운 leaf edge를 만든다(Rule 2).

- Phase 3는 extension 3가 끝남으로써 종료된다. Phase 3에서는 총 3개의 extension이 수행되었다. Rule 1은 첫 두 개의 extension에 적용되었고, Rule 2는 3번째 extension에 적용되었다.

***

- Phase 4에서는 네 번째 문자를 읽을 것이고, 최소 1번, 최대 4번의 extension이 수행된다. 우리의 예에서는 phase 4에서 네 번째 문자 'a'를 읽을 것이다. 추가될 suffix는 "abca", "bca", "ca", "a"이다. 

![Fig26]

- Extension 1에서는 suffix "abca"를 tree에 넣는다. Label "abc"를 갖는 경로가 leaf edge에서 끝나므로, 이 edge의 마지막에 'a'를 추가한다. Extension 1은 그냥 end index를 1만큼(3에서 4로) 증가시킴으로써 종료된다(Rule 1).

![Fig27]

![Fig28]

- Extension 2와 3도 유사하게 Rule 1이 적용된다...

- Extension 4는 suffix "a"를 tree에 넣는다. 이 때, label 'a'를 갖는 경로는 이미 tree에 존재한다. 아무 일도 하지 않아도 되며, Phase 4가 즉시 종료된다(Rule 3, Trick 2). 이것은 implicit suffix tree의 예가 된다. 여기서 suffix "a"는 tree에 explicit하게 나타나지 않는다. 즉, a를 path-label로 갖는 leaf node가 없다. 따라서 extension 4 이후에는 tree 구조에는 아무런 변화가 없는 것이다(Figure 28과 동일). Phase 4는 extension 4에서 Rule 3가 적용되는 순간 종료된다. Phase 4에서는 4개의 extension이 수행되었다. 첫 3개의 extension에서는 Rule 1이 적용되었고, 네 번째 extension에서는 Rule 3가 적용되었다.

***

이제 몇 개의 관찰 사실에 대해 논해볼 것이며, 어떻게 이것을 구현할 것인지에 대해 다루어 볼 것이다.
1. Phase **i**의 종료 시점에, tree에는 최대 **i**개의 leaf edge가 존재한다. 

2. Phase **i**가 종료된 이후, 모든 leaf edge의 **end** indices는 **i**이다. 이것을 어떻게 구현하면 좋을까? 모든 extension에 대해서, leaf edge를 찾고, **end** index를 increment하면 될까? 답은 "아니다" 이다. 이를 위해, 우리는 하나의 전역 변수(예를 들어, END)를 정의하고, 단순히 이 전역 변수를 1만큼 증가시킴으로써 모든 leaf edge의 end indice를 1 증가시킬 것이다. 이렇게 함으로써, 만약 phase **i** 이후에 **j**개의 leaf edge가 있다면, phase **i+1** 의 첫 **j**개의 extension은 단순히 END를 1 증가시킴으로써 완료된다(END는 이 시점에 **i+1**이 된다). 여기서 우리는 **Trick 3 - 한번 leaf는 영원한 leaf**를 구현하였다. 이 trick을 사용함으로써 우리는 임의의 phase에서 모든 **j** 개의 leaf edge에 대해 Rule 1을 적용하는 것을 **constant time**에 해낼 수 있다. 한 phase 내에서 Rule 1은 첫 몇 개의 extension(1부터 j까지)에만 적용되고, 그 이후에는 오직 Rule 2와 Rule 3만이 적용된다는 사실을 상기하자.

3. 지금까지 설명한 예제에서, trick 3가 적용되지 않는 extension에 대해 우리는 root에서 시작하여 label이 매칭되는 경로를 찾아 나갔다. 만약 phase **i** 이후에 **j**개의 leaf가 있다면, phase **i+1**에서 첫 **j**개의 extension들은 Trick 3을 사용, constant-time에 완료될 수 있다. 이제 이 phase를 완료하기 위해서는 **i+1-j**개의 extension을 더 수행해야 한다. 이 extension들에 대해 우리는 어떤 node(root라던지, 다른 internal node라던지)에서 탐색을 시작해야 하며, 어떤 경로를 따라서 탐색을 해야 하는가? 이 질문에 답을 하려면 이전 phase **i**가 어떻게 종료되었는지 생각해 보아야 한다.

* 만약 이전 phase **i**에서 모든 **i**개의 extension이 수행되었다면(**i**번째 문자가 처음 보는 문자인 경우), 다음 phase **i+1**에서 trick 3를 적용하게 되면 첫 **i**개의 suffix에 대해 Rule 1을 적용한 것이 된다. 그 다음 extension **i+1**은 root node에서 시작해 단 하나의 문자 **S[i+1]** 을 Rule 2를 이용하여 tree에 추가할 것이다.

* 만약 이전 phase **i**가 조기에(extension **j**에서 Rule 3이 적용) 종료되었다면(오직 Rule 3이 적용되었을 때만 일어나는 상황 - 즉, **i**번째 문자가 이미 적어도 한번 나타난 문자인 경우), **j-1** 개의 leaf edge가 있다는 것이다. 

  지금까지 다룬 몇 가지 사실들을 정리해보자. (계속 반복되는 내용이지만, 확실히 해두고자 한다.) 
* *Phase 1 은 Rule 2를 적용함으로써 시작하고, 다른 모든 Phase는 Rule 1을 적용함으로써 시작한다.*
* *어떤 Phase던 간에 Rule 2혹은 Rule 3로 종료된다.*
* *한 Phase 내에서는 Rule 1, Rule 2, Rule 3 순서로 적용되며, 순서가 절대 섞이지 않는다. Rule 1이 p번, Rule 2가 q번, Rule 3가 r번 적용된다고 하자*
* *p + q + r <= i 이다.*
* *Phase **i**가 종료될 때, p+q개의 leaf edge가 있으며, 다음 phase **i+1**에서는 p+q개의 extension에 대해 Rule 1이 적용된다.*

  Phase **i+1**에서는 총 **j-1**개의 leaf edge에 대해 Rule 1, 즉 Trick 3이 적용되는 것을 보았다. 바로 다음의 extension **j**는 우리가 **j**번째 suffix를 넣어야 되는 자리에서 시작하게 하고 싶다. 이를 위해서, 우리는 가장 잘 일치되는 edge를 찾아야 하고, 그 edge의 마지막에 새로운 문자를 넣어야 한다. 어떻게 가장 잘 일치되는 edge의 마지막을 찾을 수 있을까? **j**번째 suffix와 일치하는 label을 갖는 경로를, root부터 시작해서 문자 별로 하나 하나 비교해가며 탐색해야 하는가? 이렇게 하면 전체 수행 시간이 선형 시간이 되지 않을 것이다. **activePoint**를 사용하면 이를 해결할 수 있다.

  이전 phase **i**의 extension **j**에서(Rule 3이 적용된), 경로 탐색은 S[j..i-1] 라벨을 가진 edge를 찾아 놓고 보니 다음 문자가 **i**번째 문자였기 때문에 Rule 3이 적용되어 끝난 것이다. 이 때, 이렇게 찾은 **i**번째 문자의 위치를 phase **i+1**의 extension **j**에서의 시작 위치로 삼으면 된다. (다시 말해, phase **i**의 extension **j**에서는 S[j..i-1]를 찾고, phase **i+1**의 extension **j**에서는 S[j..i]를 찾게 되는데, 우리는 이미 S[j..i]가 어디서 끝나는지 알고 있다!) 여기서 **activePoint**는 이렇게 이전 extension이 어디서 끝났는지에 대한 정보를 기억해 둠으로써, 불필요한 경로 탐색을 없애 준다. Rule 1이 적용되는 첫 *p*번의 extension에서는 경로 탐색이 필요가 없다. 경로 탐색은 Rule 2나 Rule 3가 적용될 때 필요해지게 되는데, 이 시점에 activePoint는 우리가 어느 지점부터 경로 탐색을 시작해야 하는지 알려준다. **경로 탐색이 필요한 extension이라면, 항상 activePoint가 이미 시작될 위치를 가리키고 있도록 구현된다(하나의 예외는 아래에서 논의될 APCFALZ 케이스이다). 그리고 이 extension의 마지막에 activePoint를 경로 탐색이 시작될 새로운 적당한 위치로 리셋하게 된다.**

  **activePoint:** Root node가 될수도 있고, internal node나 edge 중간의 어떤 점도 될 수 있다. Extension이 시작되는 위치로서 기능하게 된다. Phase 1의 첫 번째 extension의 경우 activePoint는 root로 설정된다. 다른 extension의 경우 이전 extension에서 적절히 설정된 activePoint를 얻어서 extension을 시작한다. 그리고 Rule 2나 Rule 3가 적용될 다음 extension을 위해 이번 extension에서도 적절하게 activePoint를 설정할 의무가 있다.

  이를 달성하기 위해 우리는 activePoint를 저장할 어떤 방법이 필요하다. 우리는 이를 3개의 변수에 저장할 것이다: **activeNode, activeEdge, activeLength.**

  **activeNode:** Root node나 internal node가 될 수 있다.

  **activeEdge:** 만약 우리가 Root node나 internal node상에 있고, edge를 따라 walk-down을 해야 한다면, 우리는 어떤 edge를 골라 타고 내려가야 할지 알아야 한다. activeEdge는 이에 대한 정보를 저장한다. 때때로 activeNode 그 자체가 경로 탐색의 시작점으로 활용될 수 있는데, 이 경우 activeEdge는 다음 phase에 tree에 더해질 문자로 설정된다.

  **activeLength:** activeEdge에 나타난 edge를 타고 얼마나 내려가야 하는지에 대한 정보를 저장한다. 때때로 activeNode 그 자체가 경로 탐색의 시작점으로 활용될 수 있는데, 이 경우 activeLength는 **0**으로 설정된다.

  (아래 그림에 몇 가지 예가 나타나 있다.)

  ![active_point_examples]

  Phase **i** 이후, 만약 phase **i+1**에 j개의 leaf edge가 있다면 첫 **j**개의 extension은 trick 3에 의해 처리된다. activePoint는 extension **j+1**부터 **i+1**까지 필요할 것이며, 바로 앞의 extension이 어디서 끝나느냐에 따라서 activePoint가 바뀔 수도 있고, 바뀌지 않을 수도 있다.

  **activePoint change for extension rule 3 (APCFER3):** 임의의 phase **i**에 rule 3이 적용될 때, **phase **i+1**로 넘어가기 전에 activeLength를 1만큼 증가시킨다. activeNode나 activeEdge의 변화는 없다. 왜일까?** 앞에서 설명했던 것과 같이, 현재 extension이 사용했던 activePoint는 다음 (phase **i+1**의) extension이 사용할 activePoint의 하나 이전 문자이기 때문이다.

  **activePoint change for walk down (APCFWD):** activePoint는 extension이 끝날 때, 적용된 extension rule에 따라 바뀔 수 있다. 또한 activePoint는 walk-down을 수행할 때 바뀔 수 있다. 예를 들어, 위의 activePoint를 설명한 그림에서 현재 activePoint가 (A, s, 11)이라 하자. 이것이 어떤 extension을 시작하는 위치라면, activeNode A에서 시작하여 edge를 따라 walk-down을 하는 동안 다른 internal node가 발견된다. 이렇게 다른 internal node가 발견될 때마다 activeNode를 그 node로 바꾸어준다(따라서 activeEdge와 activeLength도 덩달아 적절하게 바뀌게 된다). 위의 예에서는

  (A, s, 11) -> (B, w, 7) -> (C, a, 3)

  순으로 activePoint가 바뀌게 되며, 위 3개의 activePoint들은 모두 같은 지점 'c'를 가리킨다.  

  다른 예를 들어보자.

  만약 extension의 시작 시 activePoint가 (D, a, 11)이었다면, walk-down 하는 동안 activePoint의 변화는 아래와 같다.

  (D, a, 10) -> (E, d, 7) -> (F, f, 5) -> (G, j 1)

  위 4개의 activePoint들은 모두 같은 지점 'k'를 가리킨다.

  만약 activePoint가 (A, s, 3), (A, t, 5), (B, w, 1), (D, a, 2) 등이라면, 즉 walk-down 시 다른 internal node를 만나지 않는다면, APCFWD에 의한 activePoint의 변화는 없다. 아이디어는 간단하다: **우리가 도달하고 싶은 지점에서 가장 가까운 지점이 activePoint가 되어야 한다. 그것이 다음 extension에서 탐색 경로를 가장 줄이는 방법이기 때문이다!**

  **activePoint change for Active Length ZERO (APCFALZ):** 마찬가지로 위의 그림에서 activePoint (A, s, 0)를 생각하자. 그리고 string **S**에서 현재 추가될 문자가 'x'라고 하자. Extension의 시작 시, activeLength가 **ZERO**이면 activeEdge는 현재 추가될 문자로 설정된다(즉 'x'로), 왜냐하면 이 경우 activeLength 가 **ZERO**이므로 walk-down이 필요가 없고, 따라서 우리가 다음에 보아야 할 문자는 현재 추가될 문자가 될 것이기 때문이다.

  4\. 코드 구현 시, 우리는 string **S**의 모든 문자를 하나씩 순회할 것이다. **i**번째 문자에 대한 루프는 phase **i**를 처리할 것이다. 각 phase 내에서 루프는 1번 또는 그 이상 반복되는데, 이것은 얼마나 많은 extension이 수행되어야 하는지에 따라 달려 있다(phase **i+1**에서 명시적으로 **i+1**번의 루프를 돌지는 않아도 됨을 명심하자. 우리에게는 Trick 3가 있다!). 우리는 변수 **remainingSuffixCount**를 사용하여 앞으로 얼마나 많은 extension이 명시적으로 수행되어야 하는지 기억해둘 것이다. 또한, 각 phase의 끝에 도달 시, 만약 **remainingSuffixCount**가 **0**이라면 이번 phase에서는 모든 suffix들이 tree에 집어넣어졌고 tree 내에 명시적으로 존재한다는 의미이다. 만약 **remainingSuffixCount**가 0이 아니라면 그만큼의 suffix들이 명시적으로 tree에 더해지지 않았다는 것(Rule 3이 적용되어 phase가 조기에 종료되었기 때문)이다. 이러한 암묵적인 suffix들은 추후에 unique한 문자가 나타날 때 tree에 명시적으로 나타나게 될 것이다. 

### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-3/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-3/
[2]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/
[Part1]: ../Ukkonen's-suffix-tree-construction-Part-1
[Part2]: ../Ukkonen's-suffix-tree-construction-Part-2

[Fig20]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig20-300x137.jpg
[Fig21]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig21-300x144.jpg
[Fig22]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig22-300x161.jpg
[Fig23]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig22-300x161.jpg
[Fig24]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig24-300x138.jpg
[Fig25]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig25-300x159.jpg
[Fig26]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig26.jpg
[Fig27]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig27.jpg
[Fig28]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig28.jpg
[active_point_examples]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/active_point_examples-300x212.jpg