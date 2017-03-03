---
title: "Ukkonen's suffix tree construction - Part 2"
layout: post
comments: true
date: 2017-02-28
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

[Ukkonen's suffix tree construction - Part 1][2]에서 우리는 고수준 Ukkonen 알고리즘에 대해 알아보았다. 이번에 다룰 두번째 파트는 [Part 1][2]의 연장선에 있다.

길이 m인 string S에 대한 suffix tree를 만들 때, 총 **m**개의 phase가 있으며 phase j(1 <= j <= m)에 대해서 우리는 총 **j**번의 extension을 통해 j번째 문자를 지금까지 만들어 놓은 tree에 추가한다. 모든 extension은 [Part 1][2]에서 다룬 3개의 extension rule 중 하나를 따른다.

Phase i+1 의 j번째 extension을 위해서(즉, 문자 S[i+1]을 추가하기 위해서), 우리는 tree의 root으로부터 시작하여 S[j..i]로 라벨이 붙여진 경로를 찾아야 한다. 하나의 방법은 실제로 root부터 시작해서 S[j..i]에 매칭되는 edge를 traverse하는 방법이다. 이 방법을 이용해서 suffix tree를 구현하는 데는 $$ O(m^3) $$ 시간이 소요된다. 몇 개의 관찰과 구현 상의 트릭을 이용하면, 이것을 $$ O(m) $$ 시간에 구현할 수 있는데, 그 방법을 알아보기로 하자.

### Suffix Links

x를 하나의 문자, A는 공백일 수 있는 임의의 substring이라 하자. Path-label을 **xA**로 갖는 internal node v에 대해서, path-label을 **A**로 갖는 다른 node s(v)가 있을 때, v에서 s(v)로 가는 pointer를 **suffix link**라고 한다.

만약 **A**가 empty substring이라면, internal node에서 시작된 suffix link는 root node를 향하게 된다.

Root node에서 시작하는 suffix link는 없다.

![Fig15]


임의의 phase **i**의 extension **j**에서 path-label **xA**를 갖는 새로운 internal node **v**가 추가된다면, 같은 phase **i**의 extension **j+1**에서는:

- **A**로 라벨 붙여진 path는 이미 internal node에서 끝난다. (**A**가 empty substring이라면 root node에서 끝난다.)
- **혹은** substring **A**의 끝에 새로운 internal node가 추가된다.

같은 phase **i**의 extension **j+1**에서, 우리는 extension **j**에서 만들어진 internal node에서 시작하는 suffix link를 만들고, 그 suffix link를 **A**로 라벨 붙여진 node로 향하게 할 것이다.

따라서 임의의 phase 내에서, (path-label **xA**를 갖는) 새로운 internal node가 생긴다면, 바로 다음 extension이 끝나는 시점에는 해당 internal node에서 나오는 (path-label **A**를 갖는 internal node를 가리키는) suffix link가 생기게 된다.

Phase **i** 이후의 임의의 implicit suffix tree에서, 만약 **xA**의 path-label을 갖는 internal node가 존재한다면, path-label **A**를 갖는 node s(v) 또한 존재하며, v에서 나오는 suffix link는 s(v)를 가리키게 될 것이다.

임의의 시점에, 지금 막 생긴 internal node를 제외한 모든 internal node는 다른 internal node(혹은 root)를 가리키는 suffix link를 가진다. 지금 생긴 internal node의 경우는 바로 다음 extension이 끝나는 시점에 자신으로부터 나오는 suffix link를 가지게 될 것이다.

### 어떻게 suffix link로 시간을 단축시키는가?

Phase **i+1**의 extension **j**에서, 우리는 현 상태의 tree의 root에서 시작하여 S[j..i]로 라벨 붙여진 경로의 끝부분을 찾아야 한다는 것을 앞에서 언급하였다. 한 가지 방법은 root에서 시작하여 S[j..i] string에 매칭되는 edge를 탐색해나가는 것이다. 이 때, suffix link는 이 경로의 끝 부분을 찾는 지름길을 제공한다!

![Fig16]

![Fig17]

위의 그림들에서 볼 수 있듯이, 우리는 경로 **S[j..i]** 를 root로부터 찾을 필요가 없다! 

**바로 앞 extension에서 S[j-1..i]의 끝 부분을 찾아 놓았다면, 거기로부터 시작해서 하나의 edge를 타고 올라가서 node v를 찾고(이 때, 타고 올라간 edge의 label을 y라고 하자. 그림 상에서는 abcd가 y이다.), v의 suffix link를 타고 s(v)로 간 다음, y에 해당하는 경로를 찾아 아래로 내려가면, 그곳이 바로 S[j..i]의 끝부분이 된다.**

Suffix link 방법을 사용하면 매번 root에서부터 경로를 찾을 필요가 없어지게 되어 수행 시간의 이득을 볼 수 있음을 알 수 있다.

**Note:** 다음 Part 3에서는 하나의 edge를 타고 올라가는 walk-up이 필요가 없어지도록 하는 **activePoint** 개념에 대해 알아볼 것이다. 이것을 이용하면 곧바로 v에서 s(v)로 갈 수 있게 된다.

**v**에서 시작하여 **s(v)** 를 향하는 suffix link가 있을 때, v에서 시작하여 leaf node를 잇는 경로가 존재하여 label **y**를 가진다면, s(v)에서 시작하여 leaf node를 잇는 경로 중 label **y**를 갖는 경로가 반드시 존재한다. Fig.17에서, v에서 leaf를 잇는 경로 중 label **abcd**를 갖는 경로가 존재하며, s(v)에서 leaf를 잇는 경로 중에도 label **abcd**를 갖는 경로가 존재함을 알 수 있다.

이 사실을 이용하면 **s(v)** 에서 경로 **y**를 따라 leaf로 이동하는 과정을 개선할 수 있다. 이를 **skip/count trick**이라 한다.

### Skip/Count Trick

**s(v)** 에서 leaf로 이동할 때, 문자 하나 하나 매칭되는지 확인할 필요가 없이, 어떤 edge를 타고 이동할 것인지 시작 문자만 확인한다면 해당하는 문자의 갯수만큼 skip해가며 다음 node로 이동하면 된다. (하나의 node에서 나오는 edge들의 시작 문자는 각각 유일하고, 경로 **y**가 존재함을 이미 알고 있기 때문) 만약 edge상의 문자 수가 이동해야 할, 즉 남은 **y**의 문자 수보다 많다면 해당 edge 상의 **y**의 마지막 문자 위치로 바로 skip한다. 

만약 edge의 label 길이와, string S의 주어진 위치의 문자를 constant time에 얻을 수 있게 구현한다면, skip/count trick은 **s(v)** 에서 leaf 까지의 walk-down 시간을 **y**의 문자열 길이가 아니라, 경로 상의 node 개수에 비례하도록 개선한다.

![Fig18]

Suffix link와 skip/count trick을 이용하면, 각각이 $$ O(m) $$ 시간이 소요되는 m개의 phase가 존재하여 suffix tree는 $$ O(m^2) $$ 시간에 만들어진다.

### Edge-label compression

여태까지 path-label은 string 안의 문자들로 표현하였다. 이렇게 suffix tree를 만들게 되면 이런 작은 string들을 저장하기 위해서 $$ O(m^2) $$ 만큼의 공간이 필요하다. 이를 피하기 위해 우리는 각 edge마다 (start, end)로 표현되는 한 쌍의 indices를 저장한다. 각각의 index는 edge label이 string S의 어디에서 시작하여 어디에서 끝나는 substring인지 나타낸다. 이렇게 구현할 때 suffix tree는 $$ O(m) $$ 공간이 필요하다.

![Fig19]

Extension rule이 바로 이어지는 extension 혹은 phase들과 맺는 관계에 대한 사실이 두 가지 더 있다. 이러한 관찰을 바탕으로 만들어진 구현 트릭 두 가지에 대해 더 알아보자. (첫 번째는 "skip/count trick" 이었다.)

### Observation 1: Rule 3은 show-stopper

Phase **i**에는 총 **i**번의 extension이 행해져야 한다.

만약 phase **i+1**의 extension **j**가 Rule 3이 적용되는 extension이라고 하자. 즉, **S[j..i]** 의 라벨을 갖는 경로 바로 다음 문자가 **S[i+1]** 이어서 이번 extension에서는 아무 일도 하지 않아도 되는 경우라고 하자. 

**이 때, 이번 phase 내에서 extension j 이후로 이어지는 모든 extension들은 Rule 3이 적용된다.** 다시 말해, phase **i+1**의 extension **j+1**부터 **i+1**까지의 모든 extension들에는 Rule 3이 적용된다.

왜 그런지 알아보자. **S[j..i]** 의 라벨을 갖는 경로 바로 다음 문자가 **S[i+1]** 임을 우리는 알고 있다. 이 때, 라벨 **S[j+1..i], S[j+2..i], S[j+3..i], ..., S[i..i]** 를 갖는 경로들 뒤에는 모두 **S[i+1]** 이 이어진다. ([Part 1][Part1]의 Figure 11, Figure 12, Figure 13 참조)

Figure 11에서 **xab**가 tree에 들어갔다. 그리고 Figure 12에서는 다음 문자 **x**를 더하려고 하고 있다. 여기서 3번의 extension이 진행되었고, 마지막 extension에서는, 마지막 suffix **x**가 이미 tree상에 존재하므로 Rule 3이 적용되었다.

Figure 13에서 우리는 문자 **a**를 tree에 더하고자 한다. 첫 3번의 suffix들 (xabxa, abxa, bxa)는 경로의 끝에 a를 더함으로써 완료되었고, 마지막 2개의 suffix들 (xa, a)의 경우 xa와 a가 이미 tree에 존재하므로 Rule 3이 적용되었다. 이는 suffix **S[j..i]** 가 이미 tree에 존재한다면, 나머지 suffix들 **S[j+1..i], S[j+2..i], ..., S[i..i]** 또한 반드시 tree에 존재할 것이므로, 나머지 suffix들에 대해서는 아무 일도 하지 않아도 된다는 것을 보여준다.

(즉 S[j..i]가 tree에 존재한다는 것은 이전 phase를 거쳐 오면서 어떻게든 S[j..i]를 tree에 집어 넣었다는 것이고, extension이 이루어지는 방식을 잘 생각해보면, S[j..i]를 집어 넣었다는 것은 S[j+1..i], S[j+2..i], ..., S[i..i]를 tree에 넣었다고 할 수 있는 것이다.)

![Fig11]

![Fig12]

![Fig13]

따라서 **Rule 3이 적용된 이후의 extension에 대해서는 생각할 필요가 없다.** 만약 새로운 internal node **v**가 extension **j**에서 생성되고, 다음 extension **j+1**에서 rule 3이 적용된다면, 우리는 node **v**로부터 현재 노드(현재 internal node에 있다면) 혹은 root node로 향하는 suffix link를 추가한다. 

### Trick 2

Rule 3이 적용되는 즉시 해당 phase를 종료한다. 남은 extension들은 암묵적으로 이미 tree에 표현되어 있기 때문이다.

### Observation 2: 한번 leaf는 영원한 leaf

만약 leaf가 만들어지고, **j**로 라벨이 붙여져 있다면(즉 string S의 position j에서 시작하는 suffix를 나타낸다면), 이 leaf는 앞으로 나타날 phase와 extension 상에서 영원히 leaf로 남을 것이다. 한 번 leaf가 **j**로 라벨 붙여지고 나면, 이후에 나타나는 모든 phase에 대해서 extension **j**에서는 항상 Rule 1이 적용될 것이다.

[Part 1][Part1]의 Figure 9 ~ 14를 참조하자.

![Fig9]

![Fig10]

**Figure 10**에서 1로 라벨 붙여진 leaf에 Rule 1이 적용된다. 이 이후에 나타나는 모든 phase에서 이 leaf에는 항상 Rule 1이 적용된다.

![Fig11]

**Figure 11**에서 2로 라벨 붙여진 leaf에 Rule 1이 적용된다. 이 이후에 나타나는 모든 phase에서 이 leaf에는 항상 Rule 1이 적용된다.

![Fig12]

**Figure 12**에서 3으로 라벨 붙여진 leaf에 Rule 1이 적용된다. 이 이후에 나타나는 모든 phase에서 이 leaf에는 항상 Rule 1이 적용된다.

![Fig13]

![Fig14]

임의의 phase **i**에서, 처음 몇 개의 extension에서는 Rule 1 혹은 Rule 2가 적용되고, Rule 3이 적용되는 즉시 phase **i**는 종료된다.

또한 Rule 2는 항상 새로운 leaf를 형성한다. (가끔 internal node를 형성하기도 한다.)

만약 $$ J_i $$를 phase **i**에서 Rule 1또는 Rule 2가 적용된 마지막 extension이라고 하면(즉, **i**번째 phase 이후에는 $$ 1, 2, 3, ..., J_i $$로 라벨 붙여진 $$ J_i $$개의 leaf가 존재한다.), $$ J_i \leq  J_{i+1} $$ 이다. 다시 말해, 각 phase는 이전에 만들어진 leaf에 Rule 1을 적용하거나, 새로운 leaf을 만든다는 것이다. 

따라서 phase **i+1**에서는 extension 1에서 $$ J_i $$ 까지 Rule 1을 적용하며(아래에 설명된 Trick 3을 이용하면 아주 간단하게 해결된다.), extension $$ J_{i+1} $$ 이후로는 Rule 2가 0번 또는 그 이상 적용될 것이며, 마침내 Rule 3이 적용되어서 phase가 끝나게 된다. 따라서 각 phase의 extension에는 Rule들이 **순서대로** 적용된다! 

Edge label을 한 쌍의 indices (start, end)로 표현하고 있으므로, 임의의 leaf edge에 대해서 **end**의 값은 무조건 phase number **i**와 같게 된다. 즉, phase **i**의 leaf edge는 모두 **end = i**이며, phase **i+1**의 leaf edge는 모두 **end = i+1**이다. 

### Trick 3

Global index **e**를 두고, **e**는 항상 phase number와 동일하게 한다. 따라서 leaf edge는 (p, e), (q, e), (r, e).. 와 같은 label을 갖게 되며, 임의의 phase에서 e를 1만큼 증가시키기만 하면 모든 leaf edge에 Rule 1을 적용한 것과 동일한 효과를 얻는다.

마침내, Trick 1, 2, 3을 적용하여 suffix tree를 만들면 $$ O(m) $$ 시간이 소요된다.

단순히 문자열에 대해서 위의 알고리즘을 적용하여 suffix tree를 만들면, implicit tree가 된다(즉, 어떤 suffix가 다른 suffix의 prefix인 경우 한 suffix가 다른 suffix를 포함함으로써 tree 상에 암묵적으로 남게 된다.) 이때 우리는 terminal symbol **$**를 먼저 문자열의 끝에 더한 뒤에 알고리즘을 수행하여 explicit suffix tree를 얻을 수 있다.

여기까지, 우리는 Ukkonen 알고리즘을 사용하여 suffix tree를 구현하는 데 이용되는 대부분의 개념들에 대해 알아보았다. Part 3에서는 string **S = abcabxabcd**를 예로 들어 위의 개념들을 이용하여 차근차근 suffix tree를 만들어 볼 것이다. Tree를 만드는 도중, 구현에 도움이 되는 몇 가지 사실들에 대해 다룰 것이다. 


### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-2/
[2]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-1/

[Part1]: ../Ukkonen's-suffix-tree-construction-Part-1

[Fig9]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig9.jpg
[Fig10]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig10-300x185.jpg
[Fig11]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig111-300x198.jpg
[Fig12]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig12-300x181.jpg
[Fig13]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig13-300x160.jpg
[Fig14]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig14-300x176.jpg
[Fig15]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig15.jpg
[Fig16]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig16-300x287.jpg
[Fig17]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig17-300x288.jpg
[Fig18]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig18-297x300.jpg
[Fig19]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig19-300x147.jpg