---
layout: post
title: Aho Corasick 알고리즘
date: 2022-04-10 19:46:00 +0900
categories: Algorithm
tags: Aho_Corasick 아호_코라식 문자열
published: true
---

## 문자열 검색

문자열 검색은 프로그램에서 굉장히 중요하다. 문자열 검색을 빠르게, 정확하게 하는 일은 프로그램의 핵심적인 동작이 되기도 한다. 그 일반적인 필요때문에, 문자열을 탐색하는 많은 알고리즘이 있다. 그 중 오늘은 하나의 문자열에서 여러 목표 패턴을 동시에 찾을 수 있게 해주는 Aho Corasick알고리즘에 대해서 기록해본다.

## 이 글의 내용에 앞서서

인터넷에 다른 글들이 많기도 하고, 긴 글을 다시 적기에는 큰 의미가 없다고 느껴진다. 이 글을 읽게 될 한국인 독자들이 여러 글을 찾아보는 것으로 아마 자신이 이해하기 쉬운 글을 스스로 찾을 것이라 생각한다. 필자도 많은 글을 영어권과 한국어권에서 찾아보았으나, 그 설명의 상세함으로 스텐포드 대학교의 대학 강의교재로 만들어진 것으로 보이는 아래의 파일을 추천한다. <https://web.stanford.edu/class/archive/cs/cs166/cs166.1166/lectures/02/Small02.pdf>

오늘 다루는 내용도 상당부분 위 PDF파일에서 얻어온 것이다.

## Aho Corasick 알고리즘

이 알고리즘은 개발자인 알프레드 아호와 마가렛 코라식의 이름을 따서 지어졌다. 이 알고리즘은 KMP알고리즘의 여러 단어 버전처럼 작동한다고 한다. 전하는 말처럼 적는 이유는 보통 권장하는 순서와 달리 KMP를 배우지 않고 아호 코라식 알고리즘을 배우고 이 글을 적고 있기 때문이다. 아호 코라식 알고리즘은 찾을 단어들의 전체 길이 M에 검색 대상이 되는 문자열의 길이인 N을 더한 수행시간을 가진다고 한다.

### 트리 구조에 대해서

아호 코라식 알고리즘은 트리 구조로 찾을 목표 단어들의 자료를 만들게 된다. 흔히 Trie(고정된 종류의 자식만을 가지는 Tree의 한 종류)로 구현된 예제를 많이 볼 수 있는데, 알고리즘의 본질적인 내용이라거나 수행 시간을 결정짓는 내용은 아니다.

아무튼 트리 구조로 만들면 이것이 유한 상태 기계(Finite State Automata)라는 것을 알 수 있다. 우리는 검색 문자열의 각 문자들을 가지고 이 트리를 돌아다니면서 결과를 얻으면 되는데, 남은 것은 어떻게 돌아다닐까와 어떻게 결과를 얻어낼까이다.

### 어떻게 돌아다닐까

아래와 같은 모양의 트리가 있다고 해 보자.
```text
■┳A`━B`━C`━D━E`
 ┗B ┳C ━D`━F`
 ┃  ┗G`
 ┗C ━D ━E`━F ━G`
```
왼쪽의 작은 검정 네모는 루트를 의미하고, 나머지 검은 선 글자는 트리의 간선을 의미한다. 각 글자는 노드를 의미하며, 노드 옆의 `는 해당 노드가 출력해야 할 단어를 가진 노드임을 의미한다. 출력단어는 루트로부터 해당 노드까지 오는 경로의 모든 글자들을 이어붙인 값이다.

돌아다니는 방식은 아래와 같다.

```text
루트에서 시작한다.
검색대상 텍스트의 매 글자마다 반복:
  현재 트리 위치에서 글자에 해당하는 자식이 있다면 그 곳으로 이동
  아니라면 실패 처리
```

### 실패처리란 어떻게 해야할까?

위 그림에서 ABCD까지 갔다면, 실패했을 때에는 어떻게 해야할까? 그리고 그에 앞서 실패라는 것은 무슨 뜻일까? 실패라는 것은 현재 검색중인 접두사로는 더 이상 찾을 것이 없다는 의미다. 즉 앞에서 보인 트리는 접두사 트리라고도 할 수 있다.

내가 ABCD라는 자리까지 이동해있다면 내가 처리한 접두사는 ABCD이며 여기서 현재 처리중인 글자를 발견하지 못한다는 것은 이제 다른 곳을 찾아야 한다는 의미이다.

그런데 다른 곳이라고 하면 어디를 찾아야 할까? ABCD로는 못 찾았다고 하지만, 지금까지 찾은 ABCD를 이용해서 제일 좋은 곳으로 갈 수 있다. 그것은 바로 BCD가 위치한 곳이다.

ABCD에서 딱 한 글자만 떼어낸 BCD접두사에는 내가 지금 찾는 다음 글자로 가는 가능성이 있을지도 모른다. 그러므로 이 경우 BCD의 D 이후에 내가 원하는 글자가 있는지를 보면 된다. 물론 BCD에서도 못 찾을 수 있다. 그러면 그 때에는 BCD에서 한 글자를 떼어낸 CD로 간다. 만약 CD가 없다면 D로 간다. 그런데 이 과정에서 어디로 갈지를 좀 더 효율적으로 처리할 수가 있는데 이것이 바로 실패링크이다.

### 실패 링크의 생성
실패링크를 생성하는 코드는 이렇게 되어있다.
```text
트리의 실패링크 생성하기(트리):
  트리의 루트와 루트의 직접적 자식들의 실패링크는 루트로 설정
  나머지 노드들에 대해서 BFS로 아래 함수 실행
  노드의 실패링크 생성하기(노드):
    노드의 부모의 실패링크를 A라 하자.
    A에 내가 가진 글자로 오는 경로가 있다면 그 경로가 실패링크이다.
    없다면 A의 실패링크를 다시 A로 놓고 위 과정을 반복한다.
```
이것이 인터넷에서 수 많은 다른 언어의 코드로 제시된다. 실패 링크라 함은 해당 접두사 노드(이전까지 그냥 노드라고 했지만 다른 관점에서 보기 위해서 이렇게 불러보자.)에서 최소한 앞 한 글자를 없애고 가장 길게 이어지는(한 글자 이상을 없애야 할 수도 있다.) 다른 접두사를 향하는 링크이다. 달리 말하면 접두사의 접미사라고도 볼 수 있다. 그래서 이 링크를 suffix link(접미사 링크)라고 정의하기도 한다. 루트의 접미사가 자기자신인 것은 코딩을 쉽게 한다. 1차 자식들에 대해서 실패링크가 루트인 것은 1글자뿐인 단어에서 앞의 최소 1글자를 제거하면 아무 글자도 안 남기 때문이다.

우리는 실패링크가 하나의 링크이면 충분한 것을 아는데, 왜냐하면 이 트리에서는 앞 글자 하나 혹은 n개를 지운 접미사는 루트로부터 언제나 1개의 경로만을 가지기 때문이다. 이 속성은 앞으로도 밑에서 다룰 출력 링크의 안전함을 생각할 때에도 필요하다.

깊이 우선 탐색으로 노드 실패링크를 생성하고자 하면 매 번 처리하려고 하는 노드 이전의 모든 링크들은 실패링크를 계산해서 가지고 있는 상태가 된다. 
```text
■┳A`━B`(1)━C`(3)
 ┗B(5) ━C(2) ━D`(4)
 ┗C(6)
```
위와 같은 트리가 있을 때, 루트와 루트 바로 아래의 A, B, C는 각각 실패링크가 루트로 초기화된다. 그리고 나머지는 괄호에 적은 순서대로 초기화된다.

(1)번인 B를 보자. AB까지 오고서 더 이상 찾을 글자가 없다면 앞서 말한 규칙대로 B로 가야하는데, 실패링크 처리규칙을 따르면 실제로 그렇게 됨을 알 수 있다. B의 부모인 A의 실패링크는 루트이며, 루트에서 B로 가는 경로가 있다.

(3)번 C도 적절히 초기화될 수 있을까? C의 부모인 B에는 실패링크인 다른 B(5)가 존재한다. 이것은 해당 노드의 부모 노드까지는 앞에서 1글자 이상을 제거한 접미사가 실패링크로 존재함을 의미한다.  그리고 자식의 실패링크는 늘 부모의 실패링크에서 이어지거나 새로 루트에서부터 시작된다. 그렇기에 B(5)에서 C(2)로 가는 길이 있음을 확인하고 실패링크로 저장하는 동작은 정상적이다.

C(2)가 실패링크를 찾아가면 C(6)이 나온다는 것도 우리는 알 수 있다. 여기서 보면 실패링크는 언제나 자신보다 깊이가 적은 노드를 가리키는 것을 알 수 있다.

D(4)는 C(6)에서 D로 가는 길이 있는지를 찾지만 그런 길은 없기에 D(4)의 실패링크는 루트이다. 이것을 달리 말하자면 BCD까지 찾은 상태에서 더 이상 매치가 없다면 우리는 일치하는 가능성이 있는 다른 단어를 찾기 위해서 루트에서부터 볼 수 밖에 없다는 뜻이다.

### 단어의 발견과 출력 링크
```text
■┳A━B━C`(1)━D`
 ┗B`━C(2)━F`
 ┗C`(3)
```
이런 트리가 있고, 문자열 ABC를 검색해본다고 하자. 문자열 ABC를 거쳐가면서 우리는 단어 ABC를 찾아냈다. 그런데 이것이 전부일까? 우리의 트리는 그 외에도 B, BC, C를 더 찾아냈어야 한다고 말해주고 있다. 이 단어들을 찾으려면 어떻게 해야할까? 좋은 방법이 없을까? 매 번 하나의 단어를 찾을 때마다 뒤로 돌아가서 찾아야하나? 그럼 멈추는 처리는 어떻게 하지? 더 나은 방법은 없을까? 우리가 찾아내야 했던 단어들을 실패링크로 이은 선을 그려보면 답이 나온다. 우리가 어떤 노드에 왔을 때, 그 노드 자체의 출력 뿐 아니라 그 노드로부터 갈 수 있는 모든 실패링크가 가진 출력을 다 검출해야 한다는 답이 나온다.

정말 그럴까? 그렇다. 실패링크의 다른 이름이 접미사 링크라고 했다. ABC까지 왔다면 자동으로 BC와 C는 검출된 상태임은 당연해보인다. 그리고 실제로 실패링크는 바로 이것들을 연쇄적으로 가리키게 된다.

동작으로 예를 들어 C(1)까지 갔을 때 실패링크로 C(2)에 가고, C(2)에서 실패링크로 C(3)에 가야한다. 그런데 사실 중간의 C를 빠르게 건너뛰어도 되지않을까? 우리는 실패링크를 다 따라다니는 실패시 동작이 아니라 그냥 실패링크 중에서 글자가 실제로 있는 것만을 빠르게 탐색하고 싶어한다. 그래서 출력 링크(Output Link)를 구하는 동작은 아래와 같이 이루어진다.
```text
출력링크 설정(트리):
  트리의 루트와 루트의 직속 자식들은 출력링크를 유효하지 않은 값(통상 null따위)으로 한다.
  나머지 자식들에 대해서 BFS로 아래 동작을 수행한다.
  노드 출력링크 설정(노드):
    노드의 실패링크가 출력을 가졌다면 그 노드로, 아니면 그 실패링크의 출력링크로 출력링크 설정
```
이 동작을 통해서 우리는 노드의 실패링크 중 출력을 가진 가장 가까운 것을 출력링크에 저장한다.

### 여러 부분 문자열로 이루어진 패턴

```text
■━A`━A`━A`━A`━A`
```
이런 트리가 있을 수 있다. A도 찾고 AA도 찾고...그런데 이런 경우 아래의 문자열을 검색한다면 매칭은 어떻게 될까?
```text
AAAAAAAAAA
```
보다시피 AAAAA는 중간의 아무 위치에서나 찾을 수 있고 AA도 그렇다. A는 어차피 한 글자니까 문제는 없지만 다른 모든 케이스는 1가지로 결정되어야 한다. 어떻게 결정될까?

결론적으로는 가능한 모든 것을 중복 없이 찾게 된다.

첫 글자의 위치에서 A를 찾고, 두 번째 글자 위치에서 AA와 출력링크의 A를, 세 번째 글자 위치에서 AAA와 출력 링크의 AA와 A... 이런 식으로 찾게 된다.

즉 새 글자를 통해 새롭게 찾게 되는 문자열만 결과에 추가함으로써 중복이 없게 만드는 것이다.

트리의 마지막 A에 도달한 상태에서는 문자열의 다음 A를 위해서 트리의 마지막 A의 실패링크인 네 번째 A로 이동해서 검색을 계속하게 된다는 것도 기억하자.

## 다른 주제

### 반대의 경우

오늘 알아본 아호 코라식은 상당히 강력하다. 찾을 단어들의 총 길이가 n, 검사할 문자열의 길이를 m, 단어들의 출현 횟수를 z라 하면
O(n)안에 트리 생성 및 초기화 연산이 끝나고, O(m+z)안에 모든 검색이 끝난다. 그런데 이 경우에 검사할 문자열이 아니라 찾을 단어들이 자주 바뀐다면 어떻게 해야할까? 트리 생성을 매번 수행하고 다시 검사를 처음부터 실행해야 할까? 검사할 문자열이 고정되고 찾을 단어들이 자주 바뀔 때에 반대로 O(m)시간 안에 초기화를 하고 O(n+z)만에 검사를 마치게 해 주는 알고리즘이 존재한다. 이 정반대의 특성을 지닌 알고리즘은 접미사 트리(Suffix Tree)를 사용한다. 우리가 이번에 다룬 아호 코라식이 Prefix Tree를 사용했다는 것을 생각해보면 이름부터 반대인 것이다.

### 다른 경우

찾을 패턴이 없는 경우 선형시간 이하의 수행시간을 가지고 패턴이 있다면 이차함수 시간의 수행시간을 가지는 Boyer-Moore 알고리즘이 있다. (실제로 찾을 패턴이 거의 늘 없다면 이런 알고리즘은 필요하다.)

여러 개의 패턴에 대해 동작하고 연산 수행의 상한시간이 Boyer-Moore 알고리즘과 같으면서도 실제로는 보다 더 나은 수행시간을 보여주는 Commentz-Waltz 알고리즘이 있다.

하나의 패턴에 대해 동작하는 Knuth-Morris-Pratt(흔히 KMP라고 불린다.) 알고리즘이 있다.

위의 알고리즘들은 모두 흥미롭다. 필요하다면 찾아봐도 좋다.

### 예시 코드

오늘 다룬 Aho Corasick 알고리즘으로 풀 수 있는 백준 문제 하나의 [소스코드](https://github.com/garrettjavalia/algorithm/blob/master/algorithm/BOJ/BOJ9250.ts)를 제시하면서 글을 마친다.
소스코드는 타입스크립트로 작성되었고 성능을 위한 최적화는 하지 않고 알고리즘의 범용성있는 적절한 구현만을 목표로 했다.