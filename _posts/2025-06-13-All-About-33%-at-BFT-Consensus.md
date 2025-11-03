---
layout: post
title: All About 1/3 at BFT Consensus
author: Sejin Hwang
tags: [consensus]
use_math: true
---

**TL;DR:** 전체 노드 중 비잔틴 노드의 비율이 1/3 미만임을 가정하는 것은 일반적인 BFT 합의 알고리즘이 Safety와 Liveness를 동시에 보장하기 위해 필요한 가정이다. 이러한 가정 아래서, BFT 합의 알고리즘의 처리량을 높이기 위해 전체 노드 중 일부 합의체를 선출하여 합의를 진행하는 방식은 단발적인 Global Communication을 수행하는 방식과 Global Communication 없이 합의 정족수를 상향하는 것이 있다.

<br>

# BFT 합의 알고리즘이 보장해야 하는 성질
- Safety
    - 모든 합의는 올바르게 되어야 함
    - 합의 알고리즘에 참여하는 모든 노드가 동일한 대상에 합의한다는 것을 보장한다면 Safety를 만족한다고 할 수 있음
- Liveness
    - 계속해서 멈추지 않고 합의를 해야 함
    - 합의 알고리즘에 참여하는 노드에서 장애가 발생하여도 합의가 계속됨을 보장한다면 Liveness를 만족한다고 할 수 있음

## Safety 보장 조건: Fork Attack Tolerance

## Liveness 보장 조건: Silence Attack Tolerance