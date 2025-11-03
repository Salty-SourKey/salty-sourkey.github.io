---
layout: post
title: Evolution of DAG-based BFT Consensus Algorithm
author: Sejin Hwang
tags: [consensus]
use_math: true
---

**TL;DR:** DAG-based Byzantine Fault Tolerance(BFT) 합의 알고리즘은 모든 합의 참여자가 리더로 동작함으로써 PBFT와 같은 단일 리더 합의 방식의 처리량 한계와 단일 실패 지점 문제를 극복하지만, 느린 최종성 지연 시간이라는 단점을 가진다. 이러한 DAG-based BFT 합의 알고리즘은 최종성 지연 시간을 효과적으로 줄이는 방향으로 발전하고 있으며, 3번의 메시지 교환 단계만으로 최종성이 결정되는 단일 리더 방식의 BFT 합의 방식과 비슷한 지연 시간 달성을 목표로 하고 있다.

<br>

# BFT 합의 알고리즘의 정의

## Byzantine Fault Tolerance
분산 시스템에서 발생하는 일부 구성 요소의 실패(하드웨어 고장 또는 소프트웨어 오류로 인한 서버 크래쉬) 또는 임의의 악의적인 행동에 대한 내성(정의되어 있는 프로토콜을 벗어난 모든 임의의 행동)

## 합의 알고리즘
분산 시스템에서 합의 과정의 참여자들이 단일 값에 대한 합의에 도달한다는 것(Safety), 그리고 이러한 합의가 멈추지 않고 지속해서 이루어질 수 있음(Liveness)을 보장하는 알고리즘

## BFT 합의 알고리즘
분산 시스템에서 발생하는 일부 구성 요소의 실패 또는 임의의 악의적 행동에 내성이 있는 합의 알고리즘

일반적으로 BFT 합의 알고리즘은 safety와 liveness를 모두 보장하기 위해서 다음과 가정을 사용
- 최대 f개의 장애 발생 가능 참여자가 존재할 때, 해당 BFT 합의 알고리즘의 전체 참여자 수는 3f+1 이상을 가정($n \ge 3f+1$)
- 네트워크가 유한한 기간 동안 비동기적으로 동작하지만, 비동기 기간의 종료 시점은 알 수 없다는 부분적 동기 분산 시스템을 가정(메시지 도착 시간의 상한은 보장되지 않지만, 모든 메시지는 결국에는 전달됨이 보장됨)

## 블록체인에서 BFT 합의 알고리즘이 필요한 이유
<img src="./images/2025-03-26/why_bft.png" width="100%">

블록체인이라는 분산 시스템의 참여자 모두가 동일한 데이터를 유지하기 위해서는 다음과 같은 조건들이 보장되어야 함
- 모든 참여자가 동일한 데이터에서 시작
- 각 참여자가 자신이 유지하는 데이터를 변경하는 트랜잭션을 동일한 순서로 실행 

즉, "동일한 트랜잭션 순서"에 대한 합의가 보장되어야 모든 참여자가 동일한 상태를 유지할 수 있음

<br>

# Leader-based BFT 합의 알고리즘
<img src="./images/2025-03-26/pbft.png" width="100%"> 

트랜잭션 순서 합의를 위해 참여자의 역할이 다음과 같이 나뉨
- 리더 선출 알고리즘을 통해 선정된 임의의 단일 참여자(e.g., Primary)는 트랜잭션 순서를 “제안”
- 나머지 참여자는 리더가 제안한 트랜잭션 순서에 “투표”

## Leader-based BFT 합의 알고리즘의 장단점

### 장점: Commit까지의 지연 시간이 매우 낮음
단 3번의 메시지 교환(약 300ms)만으로 트랜잭션 순서 합의의 완료를 의미하는 Commit이 완료됨

### 단점 1: 처리량의 한계
동시에 존재할 수 있는 리더는 오직 한 명이기에, 합의 알고리즘이 한 번에 처리할 수 있는 처리량은 단일 리더의 성능으로 제약됨 

### 단점 2: 단일 리더라는 단일 실패 지점
리더에서 장애 발생 시 합의는 중단되며, 다음 리더 선정으로 인한 복잡한 리더 동기화 메커니즘을 수행해야 함

<br>

# DAG-based 합의 알고리즘
<img src="./images/2025-03-26/dag.png" width="100%"> 

모든 합의 참여자가 리더로 동작하며, 합의를 위한 추가적인 메시지 교환 없이 자신이 로컬에서 유지하는 Directed Acyclic Graph(DAG)를 해석하는 것만으로 합의에 이르는 BFT 합의 알고리즘

## DAG-based BFT 합의 알고리즘의 기본적인 동작 방식
- Round라는 논리적 시계를 기반으로 동작하며, 각 참여자는 Round마다 자신이 제안할 트랜잭션 묶음을 Vertex안에 담아 다른 모든 참여자에게 전파
- 전파된 Vertex를 수신한 참여자는 해당 Vertex를 자신의 로컬 DAG에 추가
- 각 참여자는 수신한 모든 트랜잭션 묶음이 담긴 자신의 로컬 DAG를 해석하여 모든 트랜잭션의 Total Order를 결정
  - 이때, 임의의 결정론적인 방식을 통해 모든 참여자가 동일한 Total Order를 가지도록 보장해야 함

## DAG-based BFT 합의 알고리즘의 장단점

### 장점 1: Leader-based 방식보다 매우 높은 처리량
모든 참여자가 병렬적으로 Vertex를 제안 및 전파함으로써 단일 리더라는 처리량 병목 지점을 해결

### 장점 2: 단일 실패 지점 문제 해결
임의의 참여자에 장애가 발생하여도 추가적인 리더 동기화 메커니즘 없이 DAG는 계속해서 커지며 트랜잭션이 Commit됨

### 단점: Leader-based 방식보다 매우 느린 Commit까지의 지연 시간
합의 완료까지 3번의 메시지 교환 단계만 필요한 Leader-based 방식과 달리 DAG-based에서는 DAG에 속한 Vertex가 Total Order에 포함될 때까지 10번 이상의 메시지 교환이 필요

이러한 장단점을 가지고 있는 DAG-based BFT 합의 알고리즘 중 가장 기반이 되는 프로토콜인 DAG-Rider 설명을 통해 지연 시간의 문제를 서술하고, 이러한 문제를 개선해 나간 Bullshark, Shoal, 그리고 Shoal++가 높은 처리량이라는 장점을 유지하면서 단점인 지연 시간을 어떻게 줄여갔는지를 설명해 보고자 한다.

<br>

# DAG-Rider

### 문제 정의
비동기 네트워크 환경에서 Byzantine Atomic Broadcast(BAB)의 효율적 해결

BAB는 $n \ge 3f+1$일 때, 모든 정직한 참여자는 임의의 정직한 참여자가 전파한 Vertex를 반드시 수신한다는 것 그리고 동일한 순서로 Vertex를 Commit한다는 것을 보장하는 프로토콜

### 해결 방식
서로 비동기적으로 동작하는 Two-Layered 아키텍처(Communication and Consensus Layer)
- Communication Layer: 각 참여자는 정해진 규칙에 따라 Vertex 전파 및 수신을 하여 로컬 DAG를 지속적으로 구축
- Consensus Layer: 추가적인 Communication 없이 각 참여자는 자신의 로컬 DAG를 해석하여 Vertex의 Total Order를 결정

## DAG-Rider의 Communication Layer
- 매 Round마다 각 참여자는 트랜잭션 배치를 담은 Vertex를 생성 후 전파(Round 당 최대 1개)
  - Vertex v.round: Vertex v가 생성된 Round
- 각 참여자는 수신한 Vertex가 특정 조건을 만족한다면 자신의 로컬 DAG에 해당 Vertex 추가
  - DAG[r]: 참여자의 로컬 DAG에서 Round r에 해당하는 Vertex 집합
  - 로컬 DAG에 Vertex v 추가 시, DAG[v.round]에 추가됨
- 현재 Round가 r일 때, $\vert DAG[r] \vert \le 2f+1$이 된다면 다음 Round로 Advance
- 다음 Round로 넘어가는 순간, 각 참여자는 다음 Round의 Vertex를 생성 및 전파

- 각 Vertex는 Strong Edge와 Weak Edge를 가짐
- Vertex v’s Strong Edge
  - v가 생성된 Round의 바로 직전 Round에서 수신한 Vertex 집합(DAG[v.round-1])
  - Round Advance Rule에 의해 각 Vertex는 2f+1개의 Strong Edge를 가짐
- Vertex v’s Weak Edge
  - v.round-1 보다 더 이전 Round의 Vertex 중, v와 연결 경로가 존재하지 않는 Orphan Vertex 집합
  - 네트워크 속도가 느린 참여자로부터 전파되어 Strong Edge에 포함되지 못한 Vertex 또한 Total Order에 포함시키기 위해 사용
- 수신한 Vertex v가 로컬 DAG에 추가되려면 v의 Strong Edge와 Weak Edge가 로컬 DAG에 존재해야 함
  - 이를 통해 모든 정직한 참여자는 이들의 로컬 DAG에서 공통으로 가지고 있는 Vertex에 대해 동일한 Causal History(해당 Vertex와 경로가 있는 모든 선행 Vertex의 집합)를 가짐 

<subfigure>
  <img src="./images/2025-03-26/localdag.png" width="70%">
  <figcaption>참여자 4가 Round 6에서 전파한 vertex $v_2$를 수신하지 못하고 $v_3$를 전파한 참여자 1</figcaption>
</subfigure> 

## DAG-Rider의 Consensus Layer
DAG-Rider의 Consensus Layer는 추가적인 Communication 없이 로컬 DAG를 해석하여 Vertex의 Total Order를 결정한다. 이를 위해, Consensus Layer는 4개의 연속적인 Round로 구성된 Wave마다 Leader Vertex를 선출한다. 모든 정직한 참여자는 이들의 로컬 DAG에서 공통으로 가지고 있는 Vertex에 대해 동일한 Causal History를 가지기 때문에, 만약 모든 정직한 참여자가 동일한 Leader Commit Sequence를 가진다면, 모든 정직한 참여자는 동일한 Total Order를 가지게 된다. 따라서 DAG-Rider의 Leader Vertex 선출 방법은 결정론적인 방법을 사용하여 모든 참여자가 각 Wave에서 동일한 Leader Vertex를 선정할 수 있도록 한다.

### DAG-Rider의 Commit Rule
<subfigure>
  <img src="./images/2025-03-26/rider1.png" width="100%">
</subfigure> 
Commit Rule을 위해 각 참여자는 로컬 DAG를 연속된 4개 Round로 구성된 Wave로 나누며, 이때, Wave w의 k번째 Round를 round(w, k)라고 한다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/rider2.png" width="100%">
  <figcaption>Wave 1의 Leader Vertex $v_1$ 선정</figcaption>
</subfigure>
Wave의 마지막인 4번째 Round의 종료, 즉, $\vert DAG[round(w,4)] \vert \le 2f+1$이 될 시, DAG[round(w,1)] 속 Vertex 중 Wave w의 Leader v 선정한다. 이때, 만약 Leader v가 로컬 DAG에 없다면 해당 Wave는 Commit하지 않고 스킵한다. 

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/rider3.png" width="100%">
  <figcaption>Strong Path에 있는 Vertex가 하나밖에 없는 $v_1$은 Commit Rule은 만족하지 못함</figcaption>
</subfigure> 
<br>
<subfigure>
  <img src="./images/2025-03-26/rider4.png" width="100%">
  <figcaption>Round가 계속 진행됨에 따라 Wave 2의 마지막 Round 종료</figcaption>
</subfigure> 
<br>
<subfigure>
  <img src="./images/2025-03-26/rider5.png" width="100%">
  <figcaption>Wave 2의 Leader Vertex $v_2$ 선정</figcaption>
</subfigure>
<br>
<subfigure>
  <img src="./images/2025-03-26/rider6.png" width="100%">
  <figcaption>$v_2$는 Commit Rule을 만족하여 Leader Stack에 Push</figcaption>
</subfigure> 
Leader v 선정 이후, 해당 Leader와 Strong Path에 있는 DAG[round(w,4)] 속 Vertex가 2f+1개 이상이라면 v를 Leader Stack에 Push한다. 이때 Strong Path는 오직 Strong Edge만으로 구성된 Path를 의미한다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/rider7.png" width="100%">
</subfigure> 
그 후, 자신은 스킵했지만 다른 참여자가 Commit했을 수 있는 과거 Wave Leader 확인한다. 이때, **다른 참여자가 Commit한 Leader라면, 해당 Leader는 미래 Wave의 Leader와 항상 Strong Path를 가짐이 보장된다**. 따라서 과거 Wave Leader와 내 로컬 DAG에서 Commit Rule을 만족한 Leader 사이에 Strong Path가 존재한다면, 해당 과거 Leader 또한 Leader Stack에 Push한다. 이를 통해 모든 참여자가 동일한 Leader Commit Sequence를 갖도록 보장한다.


<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/rider8.png" width="100%">
</subfigure> 
이를 Total Order가 아직 결정되지 않은 Wave까지 반복하고, Leader Stack을 Pop해가며, Pop된 Leader의 Causal History 중 아직 Total Order에 포함되지 않은 Vertex를 임의의 결정론적인 순서(참여자 ID 순서)로 Commit하여 Total Order를 결정한다.

### DAG-Rider의 Safety
DAG-based BFT 합의 알고리즘의 Safety란 모든 참여자가 동일한 Vertex Total Order를 가지는 것이며, 이는 이전에 서술했듯 모든 참여자가 동일한 Leader Commit Sequence를 가지면 보장이 된다. 이를 위해 DAG-Rider는 **임의의 정직한 참여자 i가 Wave w에서 Vertex v를 Commit했다면, 모든 정직한 참여자가 Wave w’ > w에서 Commit하는 Leader Vertex u는 반드시 v와 Strong Path를 가짐**을 보장해야 한다. 해당 성질이 보장된다면, 임의의 참여자가 Commit한 특정 Wave의 Leader Vertex를 해당 Wave에서 Commit하지 못하고 넘어가도, 미래 Wave의 Leader Vertex를 Commit하는 과정에서 해당 Vertex는 임의의 정직한 참여자가 Commit했지만 자신은 스킵했던 과거 Leader Vertex와 반드시 Strong Path를 가지게 된다. 이를 통해, 이전 Wave를 확인하는 Commit Rule이 자신은 스킵했지만 다른 참여자가 Commit했을 수 있는 과거 Wave Leader를 반드시 Indirectly Commit한다. 따라서 모든 참여자가 동일한 Leader Commit Sequence를 가지게 된다.

#### 임의의 정직한 참여자 i가 Wave w에서 Vertex v를 Commit했다면, 모든 정직한 참여자가 Wave w’ > w에서 Commit하는 Leader Vertex u는 반드시 v와 Strong Path를 가짐의 증명
1. 참여자 i가 Wave w에서 Vertex v를 Commit했기에, 참여자 i의 로컬 DAG에는 v와 Strong Path에 있는 DAG[round(w, 4)] 속 Vertex가 2f+1개 이상

2. 참여자 i의 DAG[round(w, 4)] 속 Vertex와 또다른 임의의 참여자의 DAG[round(w, 4)] 속 Vertex는 최소 f+1개의 공통된 Vertex를 가짐
  - 각 Round는 2f+1개의 Vertex로 구성되며, 전체 집합의 크기가 3f+1일 때 크기가 2f+1인 부분 집합들의 교집합 크기는 최소 f+1
  - 즉, 임의의 정직한 참여자의 DAG[round(w, 4)] 속 Vertex 중 Vertex v와 Strong Path에 있는 집합 V의 크기는 f+1 이상

3. Vertex u를 포함한 Wave w’ > w의 Vertex v’는 이전 Round Vertex 2f+1개를 Strong Edge로 가리키고 있어야 하기에, V 속 최소 하나의 Vertex와 Strong Path를 가짐
  - 전체 집합의 크기가 3f+1일 때, 크기가 2f+1인 부분 집합과 크기가 f+1인 부분 집합의 교집합 크기는 최소 1

4. 따라서, 임의의 정직한 참여자 i가 Wave w에서 Vertex v를 Commit했다면, 모든 정직한 참여자가 Wave w’ > w에서 Commit하는 Leader Vertex u는 반드시 v와 Strong Path를 최소 하나를 가짐

하지만 방금 증명한 성질은 과거 Round에서 임의의 참여자가 Commit한 Leader Vertex는 내가 Commit한 Leader Vertex와 반드시 Strong Path가 있다는 것을 보장해주는 것이다. 즉, 이 성질은 내가 Commit한 Leader Vertex와 Strong Path가 없는 과거 Leader는 어떠한 정직한 참여자에서도 Commit되지 않았음이 보장되니 스킵하는 것이 올바르단 것을 보장해 주는 것이지, 내가 Commit하는 Leader Vertex와 Strong Path가 있다고 해당 Leader Vertex가 임의의 참여자에서 Commit된 Leader Vertex라는 것을 반드시 보장해주지 않는다.
- 성립하는 명제: 임의의 참여자가 과거에 Commit → Strong Path 존재
- 성립하는 명제: Strong Path 존재 X → 어떠한 참여자도 과거에 Commit하지 않음
- 성립하지 않는 명제: Strong Path 존재 → 임의의 참여자가 과거에 Commit

<subfigure>
  <img src="./images/2025-03-26/rider7.png" width="100%">
</subfigure> 
하지만 이는 상관이 없다. 위 그림 상에서 내가 Commit할 $v_2$와 Strong Path에 있는 $v_1$이 다른 참여자가 Commit했던 말던 상관없이, 내가 Commit Rule에 의해 $v_1$을 Commit하니 다른 참여자 또한 언젠가는 $v_2$를 반드시 Commit하고 $v_2$의 Causal History에 의해 모든 참여자가 $v_1$-$v_2$라는 동일한 Leader Commit Sequence를 가지게 된다.

### DAG-Rider의 Liveness
DAG-based BFT 합의 알고리즘의 Liveness란 정직한 참여자가 전파한 Vertex는 Commit됨이 멈추지 않는 것이며, DAG-Rider는 Common Core Abstraction을 통해 이를 보장한다. Common Core Abstraction이란 비동기 네트워크 환경이라도 3번의 all-to-all 메시지 전파는 모든 정직한 참여자가 최소 2f+1개의 동일한 값을 가지도록 보장한다는 성질이다.

<subfigure>
  <img src="./images/2025-03-26/riderlive.png" width="80%">
</subfigure>

위 그림과 같이 Common Core Abstraction은 다음과 같은 과정을 통해 진행된다.
1. 참여자는 임의의 값을 전파
2. 다른 참여자들이 전파한 값 2f+1개 수신 시, 자신이 전파한 값을 포함한 수신한 값들의 집합을 다시 전파
3. 다른 참여자들이 전파한 집합 2f+1개 수신 시, 수신한 집합들의 합집합을 다시 전파
4. 다른 참여자들이 전파한 집합 2f+1개 수신 시, 수신한 집합들의 합집합 도출
5. 각 참여자가 도출한 최종 합집합 간에는 최소 2f+1개의 동일한 값이 포함되어 있음

이러한 Common Core Abstraction 과정은 DAG-Rider의 Wave 진행 방식과 동일하다. 따라서 Wave의 4개 Round 종료 시, 모든 참여자는 첫 번째 Round에서 최소 2f+1개의 공통된 Vertex를 가지게 된다. 즉, 각 참여자가 매 Wave마다 Leader Vertex를 Commit할 확률은 2/3($\frac{2f+1}{3f+1}$)이며, 큰 수의 법칙에 따라 모든 Vertex는 1의 확률로 언젠가는 Commit됨이 보장된다. 

## DAG-Rider의 한계

**실용적이지 않은 Commit 지연 시간**
<subfigure>
  <img src="./images/2025-03-26/rb.png" width="40%">
</subfigure> 
DAG-Rider는 비동기 네트워크 환경에서의 Byzantine Atomic Broadcast 문제 중 “임의의 정직한 참여자가 전파한 Vertex는 모든 정직한 참여자가 수신해야 한다는 것”을 보장하기 위해 Reliable Broadcast 사용하며, 이는 하나의 Vertex 전파를 위해 총 3번의 메시지 교환 단계를 수행한다. 즉, Vertex 하나의 전파 과정 자체가 Leader-based BFT 합의 알고리즘의 Commit 지연 시간만큼 소요된다. 따라서 DAG-Rider는 일반적인 Leader-based 합의 방식보다 수십 배 높은 지연 시간을 가진다.

## DAG-Rider의 지연 시간 계산
<subfigure>
  <img src="./images/2025-03-26/riderLatency.png" width="100%">
</subfigure>

각 Vertex마다 Commit될 때까지 소요되는 메시지 교환 단계의 개수는 위와 같다. 예를 들어, Leader Vertex $v_2$의 Direct Commit에 의해 round(1, 4)의 Vertex가 Commit되기 위해서는 총 15번(Vertex 자신이 전파되기까지 3번 메시지 교환 + $v_2$가 Commit될 때까지 4개의 Round가 진행되며, 각 Round의 Vertex 전파 과정은 3번 메시지 교환을 동반) 소요된다.

또한 Leader Indirect Commit의 경우, Direct Commit 시 소요되는 지연 시간의 상수배만큼의 지연 시간을 야기한다.

<br>

# Bullshark

### 문제 정의
비동기 네트워크 환경을 가정하여 동작함으로부터 발생하는 DAG-Rider의 높은 지연 시간

비동기 네트워크 환경에서 Liveness를 보장하기 위한 Common Core Abstraction은 Leader Vertex Commit까지 4개의 연속적인 Round를 필요로 함

### 해결 방식
실제 네트워크 환경과 더 가까운 부분적 동기 네트워크 환경을 가정하고, 이를 통해 최선의 경우 2개의 연속적인 Round만으로 Leader Vertex를 Commit할 수 있게 되어 지연 시간을 개선함

부분적 동기 네트워크
- 메시지 도착 시간의 상한은 보장되지 않지만, 모든 메시지는 결국에는 전달됨이 보장됨
- 네트워크는 비동기적으로 동작하다, 알 수 없지만 유한한 시간이 지나면 GST(Global Stabilization Time)에 도달하여, 이후 모든 메시지가 알려진 시간 경계 $\Delta$ 내에 전달되지만, 네트워크 참여자들은 GST의 발생 또는 경과 여부를 알 수 없음


## Bullshark의 Round

### Round 구조
<subfigure>
  <img src="./images/2025-03-26/bullround.png" width="80%">
</subfigure>
- 홀수 Round는 Leader Round로 Pre-defined Rule을 통해 Leader Vertex는 미리 결정되어 있음
- 짝수 Round는 Vote Round로 직전 Round의 Leader Vertex에 대한 Vote를 진행
  - Vote: 짝수 Round Vertex의 Strong Edge

### Round Advance Rule
**DAG-Rider의 Round Advance Rule**
- $\vert DAG[r] \vert \le 2f+1$인 경우

**Bullshark의 Round Advance Rule**
- Leader Round
  - Leader Vertex를 수신했으며, $\vert DAG[r] \vert \le 2f+1$인 경우 다음 Round로 진행
- Vote Round
  - 이전 Round에 존재하는 Leader Vertex와 Strong Path에 있는 Vertex를 f+1개 수신했으며, $\vert DAG[r] \vert \le 2f+1$인 경우 다음 Round로 진행

즉, 비동기 네트워크 환경을 가정하는 DAG-Rider는 Leader Vertex 수신을 보장하기 위해 4개의 연속적인 Round가 필요하지만, Bullshark는 부분 동기 네트워크의 Synchrony Period에서 보장되는 즉각적인 Leader Vertex 수신을 활용하여 Commit 지연 시간을 단축하는 것이다.

추가적으로, 악의적인 참여자가 존재할 수 있는 부분 동기 환경에서 Bullshark의 Round Advance Rule은 영원히 만족되지 않을 수 있다. 이에 Bullshark는 Liveness를 보장하기 위해 Timer를 활용하고 Round Advance 조건에 Timeout 발생 여부를 추가한다. 

## Bullshark의 Commit Rule
- Direct Commit
  - Leader Vertex에 대한 Vote가 f+1개 이상인 경우 해당 Leader Vertex Commit

- Indirect Commit
  - DAG-Rider의 Indirect Commit과 동일(Commit한 Leader Vertex와 스킵한 과거 Leader Vertex 사이에 Strong Path가 하나 이상 존재하는 경우 Commit)

<subfigure>
  <img src="./images/2025-03-26/bullcommit.png" width="100%">
</subfigure>

총 4명(f=1, n=3f+1)이 참여하는 Bullshark 중 임의의 참여자의 로컬 DAG가 위 그림과 같은 경우
1. Round 1 Leader Vertex는 Round 2의 Timeout까지 Vote를 하나밖에 받지 못했기 때문에 스킵
2. Round 3 Leader Vertex는 Round 4의 Timeout까지 아예 수신 받지 못하여 스킵
3. Round 5 Leader Vertex는 2개의 Vote를 받아 Commit Rule을 만족
4. Round 5 Leader Vertex를 Commit하기 이전에, 과거 Round를 탐색하여 자신이 스킵한 과거 Leader Vertex를 Indirectly Commit 할 수 있는지 확인
  - Round 3 Leader Vertex와는 Strong Path가 존재하지 않기 때문에 Commit Sequence에 포함하지 않음
  - Round 1 Leader Vertex와는 Strong Path가 존재하기에 Indirect Commit

## Bullshark의 Safety
DAG-Rider와 동일하게 Bullshark 또한 모든 정직한 참여자가 동일한 Leader Commit Sequence를 가지면 Safety가 보장된다. 이를 위해 Bullshark는 **임의의 정직한 참여자 i가 Round r에서 Leader Vertex v를 Direct Commit했다면, 모든 정직한 참여자가 Round r’ > r에서 Commit하는 Leader Vertex u는 반드시 v와 Strong Path를 가짐**을 보장해야 한다.

#### 임의의 정직한 참여자 i가 Round r에서 Leader Vertex v를 Direct Commit했다면, 모든 정직한 참여자가 Round r’ > r에서 Commit하는 Leader Vertex u는 반드시 v와 Strong Path를 가짐의 증명
1. Commit Rule이 요구하는 f+1개의 Vote와 각 Vertex가 가리켜야 하는 2f+1개의 Strong Edge는 최소 1개의 공통된 Vertex를 가짐
  - 전체 집합의 크기가 3f+1일 때, 크기가 2f+1인 부분 집합과 크기가 f+1인 부분 집합의 교집합 크기는 최소 1
2. 따라서 내가 Commit한 Leader Vertex와 Strong Path가 존재하지 않는 과거 Leader Vertex는 어떠한 참여자의 로컬 DAG에서도 Commit되지 않았음이 보장됨

즉, 누군가가 Direct Commit한 Leader Vertex는 반드시 Commit(Directly or Indirectly)되며, 모든 참여자는 해당 Leader Vertex의 동일한 Causal History를 가지고 있기 때문에, Indirect Commit한 Leader Vertex의 실제 Commit 여부와는 상관없이 모든 참여자는 동일한 Leader Commit Sequence를 가지게 된다.

## Bullshark의 지연 시간 계산
<subfigure>
  <img src="./images/2025-03-26/bullLatency.png" width="100%">
</subfigure>

각 Vertex마다 Commit될 때까지 소요되는 메시지 교환 단계의 개수는 위와 같다. 예를 들어, Leader Vertex는 자신의 전파 그리고 Vote를 하는 Vertex의 전파까지 총 6번의 메시지 교환만으로 Commit되지만, 나머지 Vertex는 자신을 참조하는 Leader Vertex가 Commit되어야지만 뒤이어 Commit될 수 있기에, Leader Vertex 바로 이전 Round에 위치한 Vertex는 9번, Leader Vertex와 동일한 Round에 위치한 Vertex는 12번의 메시지 교환 이후에 Commit될 수 있다.

Leader Indirect Commit의 경우의 지연 시간은 DAG-Rider와 동일하다.

결론적으로, Bullshark는 부분 동기 네트워크 가정을 통해 DAG-Rider Commit 지연 시간을 약 40% 개선하였지만, 3개의 메시지 교환만으로 Commit이 발생하는 Leader-based BFT 합의 알고리즘의 지연 시간에는 아직 미치지 못한다는 것을 확인할 수 있다. 

<br>

# Shoal

### 문제 정의
DAG-based BFT 합의 알고리즘의 Bad Leader와 Sparse Leader의 해결
- Bad Leader: 비잔틴 또는 네트워크 속도가 느린 참여자의 Vertex가 Leader Vertex가 되는 경우, 해당 Wave는 스킵되어 다음 Wave Leader의 Commit까지 기다려야 함
- Sparse Leader: Round마다 Leader Vertex가 존재하지 않기에, Leader Vertex와 멀리 떨어진 Vertex일수록 Total Order에 포함되기까지 더 오래 걸림

### 해결 방식
- Bad Leader: Leader Reputation 도입
- Sparse Leader: DAG Pipelining을 통해 매 Round마다 Leader Vertex가 존재하도록 함

## Shoal의 Pipelining
Shoal은 DAG-based BFT 합의 알고리즘의 다음과 같은 성질을 이용하여 DAG의 Pipelining을 구현한다.

**DAG-based BFT 합의 프로토콜 P가 있을 때, 모든 정직한 참여자들이 P 인스턴스 시작 전에 Round-Leader 매핑 함수 F에 동의했다면 모든 정직한 참여자는 첫 번째로 Total Order에 포함할 Leader Vertex에 대해 동의함**

이를 통해, 첫 번째로 합의된 Leader Vertex가 Commit될 시, Commit된 Leader Vertex의 다음 Round부터 새로운 프로토콜 인스턴스를 시작한다면 Total Order가 보장됨과 동시에 Round마다 Leader Vertex가 존재할 수 있다.

<br>

<subfigure>
  <img src="./images/2025-03-26/shoal1.png" width="85%">
</subfigure> 
Bullshark 합의 프로토콜 진행 중 Round 3로 넘어갈 때 Commit Rule을 확인하여 Round 1 Leader Vertex를 Commit(위 성질에 의해 모든 정직한 참여자는 언젠가는 해당 Leader Vertex를 첫 번째로 Total Order에 포함)한다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/shoal2.png" width="85%">
</subfigure> 
첫 번째로 합의되는 Leader Vertex Commit 이후, 기존에 동작하는 Bullshark 프로토콜 인스턴스를 중지하고, Commit된 Leader Vertex의 다음 Round인 Round 2부터 새로운 프로토콜 인스턴스를 시작한다. 이때, Round 2 이후 DAG만을 사용하여 인스턴스를 실행한다. 이를 통해, 새로 실행되는 Bullshark 프로토콜 인스턴스는 실제 로컬 DAG의 Round 2에 있는 Vertex가 마치 Round 1의 Vertex인 것처럼 해석되어 Round 2가 새로운 Leader Round가 되고 Leader Vertex가 선정된다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/shoal3.png" width="85%">
</subfigure>
인스턴스는 계속 실행되며, Round 4로 넘어갈 때 Commit Rule을 확인하여 Round 2 Leader Vertex를 Commit한다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/shoal4.png" width="85%">
</subfigure>
그 후, 동일하게 기존에 동작하는 Bullshark 프로토콜 인스턴스를 중지하고, Commit된 Leader Vertex의 다음 Round인 Round 3부터 새로운 프로토콜 인스턴스를 시작한다. 이때, Round 3 이후 DAG만을 사용하여 인스턴스를 실행한다. 이를 통해, 새로 실행되는 Bullshark 프로토콜 인스턴스는 실제 로컬 DAG의 Round 3에 있는 Vertex가 마치 Round 1의 Vertex인 것처럼 해석되어 Round 3가 새로운 Leader Round가 되고 Leader Vertex가 선정된다.

Shoal은 이러한 DAG Pipelining을 통해 Total Order를 보장함과 동시에, 매 Round마다 Leader Vertex가 존재할 수 있도록 한다.

## Shoal의 Leader Reputation
- Leader Vertex Commit 이후 Pipelining에 의해 새 프로토콜 인스턴스 시작 전, Commit한 Leader Vertex의 Causal History를 보고 각 참여자들이 얼마나 많은 Vertex를 자신의 로컬 DAG에 추가했는지 확인
- 더 많은 Vertex를 추가한 참여자에게 편향되어 있는 새로운 Round-Leader 매핑 함수 F를 결정론적으로 계산하고, 새 매핑 함수 F와 함께 새 프로토콜 인스턴스 시작

이를 통해, Commit 지연 시간에 영향을 주는 Bad Leader의 선정 확률을 크게 낮춤

## Shoal의 지연 시간 계산
<subfigure>
  <img src="./images/2025-03-26/shoalLatency.png" width="100%">
</subfigure>

각 Vertex마다 Commit될 때까지 소요되는 메시지 교환 단계의 개수는 위와 같다. Leader Vertex Commit까지 소요되는 메시지 교환 횟수는 Bullshark와 동일하다. 하지만 Shoal에서는 Round마다 Leader Vertex가 존재할 수 있어, 모든 non-leader Vertex의 Commit까지는 9번의 메시지 교환 단계만 소요된다.

Leader Indirect Commit의 경우의 메시지 교환 횟수는 DAG-Rider 또는 Bullshark와 동일하지만, Leader Reputation 도입으로 인해 Indirect Commit이 발생할 확률이 대폭 감소한다.

결론적으로, Shoal은 Bullshark의 Commit 지연 시간을 약 40% 개선하였고 Indirect Commit의 발생 확률 또한 개선하였지만, 3개의 메시지 교환만으로 Commit이 발생하는 Leader-based BFT 합의 알고리즘의 지연 시간에는 아직 미치지 못한다는 것을 확인할 수 있다. 

<br>

# Shoal++

### 문제 정의
<subfigure>
  <img src="./images/2025-03-26/latencydisect.png" width="100%">
</subfigure>

non-leader Vertex의 높은 Commit 지연 시간
- Anchor Commit Latency: Vertex를 참조하는 Leader Vertex가 Commit될 때까지의 지연 시간
- Anchoring Latency: Leader Vertex가 해당 Vertex를 참조할 때까지의 지연 시간

### 해결 방식
- Anchor Commit Latency: Commit Rule로 f+1개의 Certified Vote를 요구하는 것이 아닌 2f+1개의 Uncertified Vote를 요구하여 Anchor Commit Latency 개선
- Anchoring Latency: 모든 Vertex를 Leader Vertex로 두어 Anchoring Latency 제거

## Shoal++의 Faster Direct Commit Rule
<subfigure>
  <img src="./images/2025-03-26/++uncertify.png" width="50%">
</subfigure>

DAG-based BFT 합의 알고리즘이 사용하는 Reliable Broadcast에는 3번의 메시지 교환 단계가 필요하며, Vote를 하는 Vertex는 이러한 3번의 메시지 교환을 통해 Certified 상태가 된다. Shoal++는 이러한 Certified Vote f+1개를 기다리는 것과 Uncertified Vote 2f+1개를 기다리는 것을 동일하다고 보는 것으로, 1번의 Vote 메시지 전파만으로 Direct Commit을 가능하게 한다.

즉, Shoal++의 Anchor Commit Latency는 Leader Vertex 전파를 위한 3번의 메시지 교환 그리고 Uncertified Vote 전파를 위한 1번의 메시지 교환만이 필요하게 되어, 총 4번의 메시지 교환만으로 Vertex를 참조하는 Leader Vertex가 Commit된다.

이때, 네트워크가 불안정하다면 2f+1개의 Vote를 모으는 것이 오래 걸릴 수 있다. 이러한 경우 속도가 빠른 f+1개의 Party가 먼저 Vote를 Certified하는 것이 더 빠를 수 있다. 따라서 Shoal++은 기존 Direct Commit Rule 또한 백업으로 두어, 시스템은 두 개의 Commit Rule 중 먼저 만족하는 Commit Rule을 선택하여 동작하도록 한다. 하지만 Shoal의 Reputation Scheme에 의해 대부분의 Leader Vertex Commit은 Fast Direct Commit Rule을 통해 발생한다고 해당 논문은 주장한다.

## Shoal++의 More Frequent Leader
Shoal++는 모든 Vertex를 Leader Vertex로 두어 Anchoring Latency를 제거한다. 단, 모든 Vertex를 Leader Vertex로 두었을 때, 모든 정직한 참여자가 동일한 Total Order를 유지하기 위해서는 병렬적인 Leader Vertex는 반드시 Pre-defined Order에 따라 Commit되어야 한다.

<subfigure>
  <img src="./images/2025-03-26/++1.png" width="85%">
</subfigure>
예를 들어, Shoal++가 참여자의 Round Robin Order(참여자 1’s Leader Vertex Commit → 참여자 2’s Leader Vertex Commit → 참여자 3’s Leader Vertex Commit → 참여자 4’s Leader Vertex Commit → 참여자 1’s Leader Vertex Commit → …)로 Leader Vertex를 Commit할 수 있다. 하지만 만약 제 Round에서 Leader Vertex를 Commit하지 못하는 경우, 다른 Leader Vertex 또한 Commit까지 딜레이가 발생할 수 있다. 위 그림은 Round Robin Commit Rule 수행 중, Round 1에서 참여자 2의 Vertex가 투표를 하나만 받아 Commit Rule을 만족하지 못한 예시이다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/++2.png" width="85%">
</subfigure>
정상적인 경우라면 참여자 1의 Leader Vertex부터 참여자 4의 것까지 차례로 Commit됐겠지만, 참여자 2의 Leader Vertex는 Commit될 수 없어 참여자 3, 4의 Leader Vertex Commit 또한 딜레이된다.

<br>
<br>

<subfigure>
  <img src="./images/2025-03-26/++3.png" width="85%">
</subfigure>
<subfigure>
  <img src="./images/2025-03-26/++4.png" width="85%">
</subfigure>
그 후, Round 3 Leader Vertex Commit을 통해 Round 1 속 참여자 2의 Leader Vertex가 Indirectly Commit될 때, Round 1 속 참여자 3, 4의 Leader Vertex 또한 그제서야 Commit된다. 이러한 경우 Leader Commit까지 상당한 딜레이가 생기지만, Shoal의 Leader Reputation 메커니즘을 사용한다면, Indirect Commit에 의한 Leader Commit의 딜레이는 거의 발생하지 않는다고 해당 논문은 주장한다.

## Shoal++의 지연 시간 계산
<subfigure>
  <img src="./images/2025-03-26/++Latency.png" width="100%">
</subfigure>

각 Vertex마다 Commit될 때까지 소요되는 메시지 교환 단계의 개수는 위와 같다. Shoal++에서 모든 Vertex는 Leader Vertex이기에 Anchoring Latency는 없으며, Faster Direct Commit에 의해 Anchor Commit Latency는 오직 4개의 메시지 교환만이 소요되기 때문이다. 

Leader Indirect Commit의 경우의 메시지 교환 횟수는 Shoal과 동일하다.

결론적으로, Shoal++는 3번의 메시지 교환만으로 Commit이 발생하는 Leader-based BFT 합의 알고리즘의 Commit 지연 시간과 거의 근접하도록 개선했다는 것을 확인할 수 있다.

<br>

# 결론
- DAG-based BFT 합의 알고리즘은 Leader-based 방식에 비해 높은 처리량과 단일 실패 지점이 없다는 장점을 가짐
- DAG-Rider부터 Shoal++까지, DAG-based BFT 합으 알고리즘의 발전은 단점이었던 Commit 지연 시간을 크게 개선
- 하지만 해당 논문들이 사용한 가정들은 Safety에 대한 의문점을 남김
  - 네트워크는 대부분 동기적으로 동작하며 비동기 상황은 극히 드물게 발생한다는 가정
  - Byzantine Failure는 거의 발생하지 않는다는 가정
