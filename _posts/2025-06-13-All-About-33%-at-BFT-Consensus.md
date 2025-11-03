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

## Safety 보장 조건: Quorum Intersection
<subfigure>
  <img src="./images/2025-06-13/intersection.png" width="50%">
</subfigure>
**Quorum Intersection이란?**

전체 노드 개수가 n이고, 그 중 f개의 비잔틴 노드가 있을 때, 정족수 q 크기의 서로 다른 노드 집합 $Q_1$ 그리고 $Q_2$는 최소 1개의 공통되는 정상 노드를 가져야 함

위 성질이 보장된다면, 서로 다른 두 블록이 정족수 이상의 투표를 동시에 모았을 때, 두 정족수 집합은 공통되는 정상 노드의 투표를 최소 1개를 가지게 된다. 하지만 정상 노드는 서로 다른 두 개의 대상에 동시에 투표하는 행동을 하지 않기에 모순이 발생한다. 즉, 서로 다른 두 블록이 동시에 정족수 이상의 투표를 모으는 것은 불가능하다. 따라서 정직한 노드가 서로 다른 대상에 합의하는 상황은 발생하지 않아 Safety가 보장된다.

이에 따라, 정족수 q는 $q+q-n-f>0$, 즉, $\frac {n+f} {2} < q$를 만족해야 한다.

## Liveness 보장 조건: Quorum Formation
**Quorum Formation이란?**

전체 노드 개수가 n이고, 그 중 f개의 비잔틴 노드가 있을 때, 정족수 q는 정상 노드 수의 최댓값보다 작아야 함

위 성질이 보장된다면, 모든 장애 발생 노드에서 장애가 발생하더라도 나머지 정상 노드들이 정족수 이상의 투표를 모을 수 있기에, BFT 합의 알고리즘에서 블록을 제안하는 리더가 비잔틴 노드가 아니라면 Liveness가 보장된다.

이에 따라, 정족수 q는 $q \ge n-f$를 만족해야 한다.

# Why n ≥ 3f+1?
Safety 보장 조건인 Quorum Intersection을 만족하기 위한 $\frac {n+f} {2} < q$과 Liveness 보장 조건 중 하나인 Quorum Formation을 만족하기 위한 $q \ge n-f$의 부등식을 풀면 다음과 같은 식이 나온다.

<center>$\frac{n+f}{2} < q \le n-f$</center>

<center>$n+f < 2n-2f$</center>

<center>$3f < n$</center>

<center>$3f + 1 \le n$</center>

즉, 전체 노드 중 비잔틴 노드의 비율이 1/3 미만임을 가정하는 것은 일반적인 BFT 합의 알고리즘이 Safety와 Liveness를 동시에 보장하기 위해 필요한 가정이라는 것을 확인할 수 있다.

# What if n/3 ≤ f?

## Breaking Liveness when n/3 ≤ f
<subfigure>
  <img src="./images/2025-06-13/breaklive.png" width="70%">
</subfigure>

전체 노드 수가 8개일 때, 일반적인 BFT 합의 알고리즘은 최대 2개의 비잔틴 노드까지 견딜 수 있다. 최악의 경우에도 Safety와 Liveness를 동시에 보장하는 정족수는 6이기 때문이다.

<center>$\frac{n+f}{2} < q \le n-f$</center>

<center>$5 < q \le 6$</center>

하지만 $f \le \frac n 3$ 가정을 넘어서는 3개의 비잔틴 노드가 존재하고, 이들이 의도적으로 메시지를 전파하지 않는 Silence Attack을 수행할 시 정족수인 6개 이상의 투표가 모일 수 없어 Liveness가 깨진다.

## Breaking Safety when n/3 ≤ f
<subfigure>
  <img src="./images/2025-06-13/breaksafe.png" width="100%">
</subfigure>

전체 노드 수가 10개일 때, 일반적인 BFT 합의 알고리즘은 최대 3개의 비잔틴 노드까지 견딜 수 있다. 하지만 이를 넘어서는 4개의 비잔틴 노드(n1, n2, n3, n4)가 존재하고 그 중 하나가 블록을 제안하는 리더라면 정직한 노드들이 서로 다른 블록을 수락하게 만드는 Forking Attack을 가할 수 있다.

리더인 n1이 서로 다른 두 개의 블록 a 그리고 b를 두 개의 정직한 노드 집합에 각각 전파하고 모든 비잔틴 노드들이 해당하는 두 블록 모두에 동시에 투표를 한다면, 블록 a 그리고 b는 동시에 정족수를 만족할 수 있다. 이를 통해 정직한 노드들이 서로 다른 블록을 수락하게 만드는 Forking Attack을 성공시킬 수 있다.