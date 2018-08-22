---
title: "HashTable의 구조와 구현"
layout: post
date: 2018-08-05 23:12
image: 
headerImage: true
tag:
- Hash Table
- Data Structure
category: blog
author: betterFLY
description:
sitemap :
  changefreq : daily
  priority : 1.0
---

## Hash Table의 이해와 구현




방법을 모르고 맹목적으로 일을 처리하면 일은 많이 하고 하는 일은 많은데 성과가 안나온다.

HashMap이란...?

물체들이 있다.
이 물체들을 1열로 세워놓지 말고, 그룹화 시켜서 A, B 바구니에 넣고 싶다 ==> Binning(bin = 바구니)
Hash Table / Hash Map 공통으로 쓰는 이유?
Hash Table 은 '어떤 값을 넣을때 Bin 에 담는다'  판단하는 것이 전부라면,
Hash Map 의 개념은 Bin 을 결정할 때 쓰는 물체와, 실제 Bin 에 넣는 물체가 다르다는 개념이다.

예륻들어, 바코드의 경우 > 바코드의 끝 숫자가 '5' 이다 >> '5'번쨰 Bin 에 넣지만 실제 넣는 것은 물체라는 것.

Key - 바코드 / Value - 물체 => Hash Table
내가 가진 물체를 가지고 어떤 바구니에 넣어야할지 결정하는 것.

어떻게?
이 모든 결정은 숫자로 결정한다... 왜? 컴퓨터가ㅓ 갖고 있는 모든 개체는 숫자이니까.
예를들어 'A' 는 ASCII Code = 65, 'a' = 97 이다.
'B' = 66, 'b' = 98

바구니가 10개 있다면, 0~9까지 index가 있따.
A,B가 있는 경우 각각 다른 바구니에 넣어보자

65,66을 각각 다른 바구니에 넣는다면?
나누기연산 (/) 을 하자 > 65 = 5, 66 = 6 번 Bin에 넣을 수 있겠다.
같은 방법으로 Bin에 나누어 처리한다면 75번 물체는 다시 5번 Bin에 들어간다.

나중에 A를 찾고싶다면? A/10 은 '5'가 남는다 -> 5번 bin 확인 >> 존재 O

이것이 Hash Table의 개념
---

1Bin 에 1개의 물체만 가능하다면?
물체의 개수만큼 배열(bin)을 만들고, 물체에서 배열 수 만큼 나눠서 각각의 값을 넣으면된다.
무슨말이냐,
0~9까지가 있다고 가정하자.
Bin 역시 10개 만든다 (0~9)
1 - 9 , 2 ->




