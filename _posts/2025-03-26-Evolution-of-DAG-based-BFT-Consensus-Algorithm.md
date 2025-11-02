---
layout: post
title: Evolution of DAG-based BFT Consensus Algorithm
author: Sejin Hwang
tags: [consensus]
use_math: true
---

**TL;DR:** DAG-based Byzantine Fault Tolerance(BFT) 합의 알고리즘은 모든 합의 참여자가 리더로 동작함으로써 PBFT와 같은 단일 리더 합의 방식의 처리량 한계와 단일 실패 지점 문제를 극복하지만, 느린 최종성 지연 시간이라는 단점을 가진다. 이러한 DAG-based BFT 합의 알고리즘은 최종성 지연 시간을 효과적으로 줄이는 방향으로 발전하고 있으며, 3번의 메시지 교환 단계만으로 최종성이 결정되는 단일 리더 방식의 BFT 합의 방식과 비슷한 지연 시간 달성을 목표로 하고 있다.


# BFT 합의 알고리즘의 정의

## Byzantine Fault Tolerance
분산 시스템에서 발생하는 일부 구성 요소의 실패(하드웨어 고장 또는 소프트웨어 오류로 인한 서버 크래쉬) 또는 임의의 악의적인 행동에 대한 내성(정의되어 있는 프로토콜을 벗어난 모든 임의의 행동)

## 합의 알고리즘
분산 시스템에서 합의 과정의 참여자들이 단일 값에 대한 합의에 도달한다는 것(Safety), 그리고 이러한 합의가 멈추지 않고 지속해서 이루어질 수 있음(Liveness)을 보장하는 알고리즘

## BFT 합의 알고리즘
분산 시스템에서 발생하는 일부 구성 요소의 실패 또는 임의의 악의적 행동에 내성이 있는 합의 알고리즘

일반적으로 BFT 합의 알고리즘은 safety와 liveness를 모두 보장하기 위해서 다음과 가정을 사용
- 최대 f개의 장애 발생 가능 참여자가 존재할 때, 해당 BFT 합의 알고리즘의 전체 참여자 수는 3f+1 이상을 가정($n \ge 3f+1$)
- 네트워크가 유한한 기간 동안 비동기적으로 동작하지만, 비동기 기간의 종료 시점은 알 수 없다는 부분적 동기 분산 시스템을 가정(메시지 도착 시간의 상한은 보장되지 않지만,모든 메시지는 결국에는 전달됨이 보장됨)

## 블록체인에서 BFT 합의 알고리즘이 필요한 이유
<img src="./images/2025-03-26/why_bft.png" width="100%">

블록체인이라는 분산 시스템의 참여자 모두가 동일한 데이터를 유지하기 위해서는 다음과 같은 조건들이 보장되어야 함
- 모든 참여자가 동일한 데이터에서 시작
- 각 참여자가 자신이 유지하는 데이터를 변경하는 트랜잭션을 동일한 순서로 실행 

즉, "동일한 트랜잭션 순서"에 대한 합의가 보장되어야 모든 참여자가 동일한 상태를 유지할 수 있음

# Leader-based BFT 합의 알고리즘
<img src="./images/2025-03-26/pbft.png" width="100%"> 

트랜잭션 순서 합의를 위해 참여자의 역할이 다음과 같이 나뉨
- 리더 선출 알고리즘을 통해 선정된 임의의 단일 참여자(e.g., Primary)는 트랜잭션 순서를 “제안”
- 나머지 참여자는 리더가 제안한 트랜잭션 순서에 “투표”

## Leader-based BFT 합의 알고리즘의 장단점

### 장점: 트랜잭션 순서의 합의 완료를 나타내는 Commit까지의 지연 시간이 매우 낮음
단 3번의 메시지 교환(약 300ms)만으로 Commit이 완료됨

### 단점 1: 동시에 존재할 수 있는 리더는 오직 한 명이기에, 합의 알고리즘이 한 번에 처리할 수 있는 처리량은 단일 리더의 성능으로 제약됨 

### 단점 2: 단일 리더라는 단일 실패 지점이 존재함(리더에서 장애 발생 시, 다음 리더 선정으로 인한 복잡한 리더 동기화 메커니즘을 수행해야 함)

# DAG-based 합의 알고리즘
<img src="./images/2025-03-26/dag.png" width="100%"> 

모든 합의 참여자가 리더로 동작하며, 합의를 위한 추가적인 메시지 교환 없이 자신이 로컬에서 유지하는 Directed Acyclic Graph(DAG)를 해석하는 것만으로 합의에 이르는 BFT 합의 알고리즘

## DAG-based BFT 합의 알고리즘의 기본적인 동작 방식
- Round라는 논리적 시계를 기반으로 동작하며, 각 참여자는 매 Round마다 자신이 제안할 트랜잭션 묶음을 Vertex안에 담아 다른 모든 참여자에게 전파
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

# DAG-Rider

### 문제 정의: 비동기 네트워크 환경에서 Byzantine Atomic Broadcast(BAB)의 효율적 해결
BAB는 $n \ge 3f+1$일 때, 모든 정직한 참여자는 임의의 정직한 참여자가 전파한 Vertex를 반드시 수신한다는 것 그리고 동일한 순서로 Vertex를 Commit한다는 것을 보장하는 프로토콜

### 해결 방식: 서로 비동기적으로 동작하는 Two-Layered 아키텍처(Communication and Consensus Layer)
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
  <img src="./images/2025-03-26/localdag.png" width="100%">
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

하지만 방금 증명한 성질은 과거 Round에서 임의의 참여자가 Commit한 Leader Vertex는 내가 Commit한 Leader Vertex와 반드시 Strong Path가 있다는 것을 보장해주는 것이다. 즉, 이 성질은 내가 Commit한 Leader Vertex와 Strong Path가 없는 과거 Leader는 어떠한 정직한 참여자에서도 Commit되지 않았음이 보장되니 스킵하는 것이 맞다는 것을 보장해 주는 것이지, 내가 Commit하는 Leader Vertex와 Strong Path가 있다고 해당 Leader Vertex가 임의의 참여자에서 Commit된 Leader Vertex라는 것을 반드시 보장해주지 않는다.
- 성립하는 명제: 임의의 참여자가 과거에 Commit → Strong Path 존재
- 성립하는 명제: Strong Path 존재 X → 어떠한 참여자도 과거에 Commit하지 않음
- 성립하지 않는 명제: Strong Path 존재 → 임의의 참여자가 과거에 Commit

<subfigure>
  <img src="./images/2025-03-26/rider7.png" width="100%">
</subfigure> 
하지만 이는 상관이 없다. 위 그림 상에서 내가 Commit할 $v_2$와 Strong Path에 있는 $v_1$이 다른 참여자가 Commit했던 말던 상관 없이, 내가 Commit Rule에 의해 $v_1$을 Commit하니 다른 참여자 또한 언젠가는 $v_2$를 반드시 Commit하고 $v_2$의 Causal History에 의해 모든 참여자가 $v_1$-$v_2$라는 동일한 Leader Commit Sequence를 가지게 된다.

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