---
title: "Ukkonen's suffix tree construction - Part 1"
layout: post
comments: true
date: 2017-02-28
categories: algorithm
---
*[geeksforgeeks Anurag Singh의 Ukkonen's Suffix Tree Constructon][1]을 번역하여 정리하였다.*

Suffix Tree는 무수한 문자열 처리와 계산생물학 문제들에 유용하게 사용될 수 있다. 많은 책들과 온라인 자료들이 이론적으로 Suffix Tree에 대해 다루고는 있고, 몇몇 곳에서는 코드 구현 또한 다루고 있다. 하지만 나는 이러한 자료들에 대해 뭔가 부족함을 느꼈고, 코드 구현과 응용이 쉽지 않음을 깨달았다. 이 글은 이론적인 배경과 잘 작동하는 코드 구현 사이의 간극을 메우기 위해 작성된 글이다. 여기서 우리는 Ukkonen's Suffix Tree Construction 알고리즘에 대해 다룰 것이다. 우리는 단계 별로, 이론과 구현의 상세한 여러 부분들에 대해 검토해볼 것이다. 우선 Brute-force한 방법으로 suffix tree를 만드는 데서부터 시작하여, Ukkonen의 알고리즘에 사용된 여러 트릭들과 같은 여러 개념들을 배우게 되며, 마지막에는 코드 구현까지 다루어 볼 생각이다.

**Note:** 알고리즘의 몇몇 부분들은 이 글을 첫번째, 혹은 두번째 읽는 동안에는 이해하기 어려울 수도 있다. 그리고 이건 당연히 괜찮다! 조금만 더 노력하고, 생각해본다면 결국에는 이해할 수 있을 것이다.

**Dan Gusfield**의 책 [Algorithms on Strings, Trees and Sequences: Computer Science and Computational Biology][2]는 Suffix Tree의 개념들에 대해 아주 잘 설명하고 있다.


m-character string S에 대한 suffix tree **T**는 rooted directed tree이며 1부터 **m**까지 번호 붙여진, 정확히 m개의 leaf를 갖는다. (String의 마지막 문자가 unique character일 때)

- Root는 0개, 혹은 1개 이상의 children을 가질 수 있다.
- Root가 아닌 각각의 internal node는 적어도 2개의 children을 갖는다.
- 각각의 edge는 S의 nonempty substring으로 라벨이 붙여진다.
- 같은 node에서 나오는 두 edge는 같은 character로 시작하는 edge-라벨을 가질 수 없다.

Root부터 leaf까지의 path를 따라서 edge-라벨을 이어붙이면, i번째 위치에서 시작하는 S의 suffix를 얻는다. 즉, S[i..m]을 얻는다.

**Note:** Index는 1부터 시작한다. (0부터 시작하지 않는다. 하지만 코드 구현에서는 0부터 시작하는 indexing을 사용하게 된다.)

String S = xabxac와 m = 6에 대해 suffix tree는 아래와 같이 나타난다.

![Fig1][Fig1]


하나의 root node와 2개의 internal node를 가지며 6개의 leaf node를 가지는 것을 확인할 수 있다.

- <span style="color:red">빨간색</span> 경로의 깊이는 1이며, 위치 6에서 시작하는 suffix c를 나타낸다.
- <span style="color:blue">파란색</span> 경로의 깊이는 4이며, 위치 3에서 시작하는 suffix bxca를 나타낸다.

- <span style="color:green">초록색</span> 경로의 깊이는 2이며, 위치 5에서 시작하는 suffix ac를 나타낸다.

- <span style="color:orange">주황색</span> 경로의 깊이는 6이며, 위치 1에서 시작하는 suffix xabxac를 나타낸다.

라벨 a(<span style="color:green">초록색</span>)와 xa(<span style="color:orange">주황색</span>)를 갖는 edge는 non-leaf edge(internal node에서 끝나는)이다. 모든 다른 edge는 leaf edge(leaf에서 끝나는)이다.

만약 S의 한 suffix가, 다른 suffix의 prefix와 일치한다면,(string의 마지막 문자가 unique하지 않을 때) 전자의 suffix를 나타내는 edge는 leaf에서 끝나지 않을 것이다.

String S = xabxa와 m = 5에 대해서 suffix tree는 아래와 같이 나타난다.

![Fig2]

5개의 suffix가 있다: xabxa, abxa, bxa, xa, a.

Suffix 'xa'와 'a'에 대한 경로는 leaf에서 끝나지 않는다. 위와 같은 (Figure 2) tree에서는 몇몇 suffix들이 tree안에 명료하게 나타나지 않기 때문에, 이러한 tree를 **implicit suffix tree**라고 부른다. 

이러한 문제를 피하기 위해서 string에 나타나지 않는 문자를 하나 추가한다. 보통 $나 # 등을 말단 문자로 사용한다.

아래는 S = xabxa$와 m = 6에 대한 suffix tree이며, 모든 6개의 suffix가 leaf에서 끝나는 것을 확인할 수 있다.

![Fig3]

### Suffix tree를 만드는 나이브한 방법

String S와 그 길이 m이 주어졌을 때, suffix를 나타내는 edge S[1..m]$ (string 전부) 를 tree에 집어넣고, 2 부터 m까지의 i에 대해서 S[i..m]$를 계속하여 집어넣는다. $$ N_i $$를 1부터 **i**까지의 suffix들을 현재 갖고 있는 중간 단계의 tree라고 하자. 그러면 $$ N_{i+1} $$은 $$ N_i $$로부터 아래와 같이 만들어진다:

- $$ N_i $$의 root로부터 시작한다.
- S[i+1..m]$의 prefix와 일치하는, root로부터의 가장 긴 경로를 찾는다.
- 만약 이 경로가 어떤 edge (u, v)의 중간에서 끝난다면, 일치하는 마지막 문자 뒤에, 다시 말해 일치하지 않는 첫 번째 문자 앞에 새로운 node w를 삽입하여 edge (u, v)를 두 개의 edge로 쪼갠다. 새로운 edge (u, w)는 이제 S[i+1..m]과 일치했던 edge (u, v) 라벨의 일부분으로 라벨 붙여지며, 새로운 edge (w, v)는 edge (u, v) 라벨의 나머지 부분으로 라벨 붙여진다.
-  w로부터 나오는 새로운 edge (w, i+1)을 만들고, 새로운 leaf는 i+1의 라벨을 갖게 한다. 새로운 edge (w, i+1)의 경우에는 앞에서 일치되고 남은 S[i+1..m]의 부분으로 라벨 붙여진다.

길이 m인 string S로부터 suffix tree를 만들 때 이러한 방법을 사용하면 $$ O(m^2) $$ 시간이 소요된다. 위의 알고리즘으로 string **xabxa$** 에 해당하는 suffix tree를 만드는 과정이 아래에 소개되어 있다.
![Fig4]

![Fig5]

![Fig6]

![Fig7]

### Implicit suffix tree
Ukkonen 알고리즘으로 suffix tree를 만들 때, string S의 문자에 따라서 중간 단계의 몇몇 부분에서 implicit suffix tree를 만나게 될 것이다. Implicit suffix tree에서는 라벨에 $ (또는 # 등의 말단 문자)를 갖는 edge와 오직 하나의 edge만을 outgoing edge로 갖는 internal node가 없다.
Suffix tree S$로부터 implicit suffix tree를 얻기 위해서는,

- Tree의 모든 edge label에서 $ 문자를 삭제한다.
- Label이 없는 모든 edge를 삭제한다.
- 오직 하나의 edge만을 outgoing edge로 갖는 node를 삭제하고, edge를 합친다.

![Fig8]

### Ukkonen 알고리즘의 고수준 설명
Ukkonen 알고리즘은 길이 m인 string S의 모든 prefix S[1..i]에 대해서 implicit suffix tree를 만든다.

가장 먼저 첫 번째 문자를 이용하여 $$ T_1 $$을 만들고, 두 번째 문자를 이용하여 $$ T_2 $$를 만들고, ..., m번째 문자를 이용하여 $$ T_m $$을 만든다.

Implicit suffix tree $$ T_{i+1} $$은 implicit suffix tree $$ T_i $$을 이용하여 만들어진다.

S의 true suffix tree는 $$ T_m $$에 $ 문자를 더함으로써 만들어진다.

특정 시점에 Ukkonen 알고리즘은 지금까지 만난 문자들만을 이용하여 suffix tree를 만들기 때문에 **on-line property** 를 가진다. 이 특성은 몇몇 상황에서 유용하게 사용될 수 있다.

$$ O(m) $$시간이 소요된다.

Ukkonen 알고리즘은 m개의 **phase** 로 나뉜다. (각각의 phase마다 하나의 문자를 삽입한다.)

Phase i+1에는 tree $$ T_{i+1} $$이 $$ T_i $$로부터 만들어진다.

각 phase i+1은, S[1..i+1]의 i+1 suffix들에 대하여 i+1개의 **extension** 들로 나뉜다. 

Phase i+1의 extension j에서는 가장 먼저 substring S[j..i]로 라벨 붙여진, root에서부터 시작하는 경로를 찾는다.

그 다음 문자 S(i+1)을 그 끝에 추가한다. (만약 그 문자가 이미 없다면)

Phase i+1의 extension 1에서 우리는 tree에 S[1..i+1]을 집어넣을 것이다. 이 때 phase i는 이미 지나갔으므로, S[1..i]은 이미 tree에 나타나 있다. 따라서 우리는 S[i+1] 번째 문자만 추가해주면 되는 것이다. (만약 없다면)

Phase i+1의 extension 2에서 우리는 tree에 S[2..i+1]을 집어넣을 것이다. 이 때 phase i는 이미 지나갔으므로, S[2..i]은 이미 tree에 나타나 있다. 따라서 우리는 S[i+1] 번째 문자만 추가해주면 되는 것이다. (만약 없다면)

Phase i+1의 extension 3에서 우리는 tree에 S[3..i+1]을 집어넣을 것이다. 이 때 phase i는 이미 지나갔으므로, S[3..i]은 이미 tree에 나타나 있다. 따라서 우리는 S[i+1] 번째 문자만 추가해주면 되는 것이다. (만약 없다면)

Phase i+1의 extension 4에서 우리는 tree에 S[4..i+1]을 집어넣을 것이다. 이 때 phase i는 이미 지나갔으므로, S[4..i]은 이미 tree에 나타나 있다. 따라서 우리는 S[i+1] 번째 문자만 추가해주면 되는 것이다. (만약 없다면)

...

Phase i+1의 extension i+1에서 우리는 tree에 S[i+1..i+1]을 집어넣을 것이다. 이 때, S[i+1..i+1]은 단순히 하나의 문자이며, 만약 이것이 처음 보는 문자라면 tree에 나타나 있지 않을 것이다. 이 경우는 라벨 S[i+1]을 갖는 leaf edge를 추가해 주기만 하면 된다.

### 고수준 Ukkonen algorithm

```
Tree T1을 만든다.

For i from 1 to m-1 do

begin {phase i+1}

	For j from 1 to i+1

		begin {extension j}

		Root로부터 시작하여, S[j..i]로 라벨 붙여진 경로를 찾는다.
		Path 끝에 S[i+1]이 이미 있지 않다면, path에 문자 S[i+1]을 추가하여 extend시킨다.
		
	end;
end;
```

Suffix extension은 단순히, 이미 있는 suffix tree에 어떻게 새로운 문자 하나를 추가할 것인가에 대한 내용이다. 

Phase i+1의 extension j에서 알고리즘은 S[j..i]의 끝을 찾는다. (S[j..i]는 이미 phase i에서 tree에 추가되었을 것이다.) 그리고 S[j..i]를 extend하여 S[j..i+1]가 tree에 존재하도록 한다.

**3가지의 extension rule이 있다:**

**Rule 1:** 만약 S[j..i]에 해당하는 경로가 leaf edge에서 끝난다면,(즉, S[i]가 leaf edge의 마지막 문자라면) 문자 S[i+1]을 단순히 leaf edge 라벨에 추가해주기만 하면 된다.

**Rule 2:** 만약 S[j..i]에 해당하는 경로가 non-leaf edge에서 끝나고,(즉, 경로 상의 문자 S[i]의 뒤에도 문자들이 더 있다면) 또한 다음 문자가 S[i+1]이 아니라면, 라벨 S[i+1]와 suffix index j를 갖는 새로운 leaf edge를 추가한다. 또한, non-leaf edge의 중간에서 s[1..i]가 끝난다면 새로운 internal node가 생성된다.

**Rule 3:** 만약 S[j..i]에 해당하는 경로가 non-leaf edge에서 끝나고, 다음 문자가 S[i+1]이라면, 아무 일도 일어나지 않는다.

한 가지 중요한 점은 어떤 node가 주어졌을 때, 라벨의 첫 번째 문자마다 오직 하나의 outgoing edge만이 존재한다는 것이다.

아래는 string xabxac에 대해 suffix tree를 만드는 과정이다.

![Fig9]

![Fig10]

![Fig11]

![Fig12]

![Fig13]

![Fig14]

### References
[http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-1/][1]

[1]: http://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-1/
[2]: http://www.amazon.in/Algorithms-Strings-Trees-Sequences-Computational/dp/0521585198

[Fig1]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig1-300x145.jpg
[Fig2]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig2.jpg
[Fig3]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig3-300x139.jpg
[Fig4]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig4.jpg
[Fig5]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig5-300x226.jpg
[Fig6]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig6.jpg
[Fig7]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig7-300x249.jpg
[Fig8]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig8-300x192.jpg
[Fig9]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig9.jpg
[Fig10]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig10-300x185.jpg
[Fig11]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig111-300x198.jpg
[Fig12]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig12-300x181.jpg
[Fig13]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig13-300x160.jpg
[Fig14]: http://d1hyf4ir1gqw6c.cloudfront.net//wp-content/uploads/Fig14-300x176.jpg

