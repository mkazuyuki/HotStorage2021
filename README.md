# HotStorage2021

Day1 2021.07.27 GMT 1400-  
Day2 2021.07.28 GMT 1400-

Day1 セッション構成は Keynote > 前半 Flash storage > break > 後半 Distributed storage

- I/O Acceleration from the Bottom Up  
  Presentation [English](https://www.youtube.com/watch?v=9C2ESO6YvNk&t=3450s)
- Scalable but Wasteful: Current State of Replication in the Cloud  
  Presentation [English](https://www.youtube.com/watch?v=9C2ESO6YvNk&t=15265s) | [Japanese](https://youtu.be/3ll_xtXkRPE)  

  分散アルゴリズムを実行するとき、クラウドでは 最適化されたアルゴリズムが より高性能となる訳ではない、という話。

  Multi Paxos のボトルネックを解消する最適化を行った EPaxos は Multi Paxos より高いスループットが得られる。
  しかし、クラウドでの現実的な使用方法を想定して、使用するリソース量(ノード数・CPU数など)を予め決めて、その中にアルゴリズムを実行するインスタンスを複数・同数 詰め込んで性能を測定すると、Multi Paxos が EPaxos より高いスループットを発揮する、という逆転現象が起こる。
  この論文は 最適化による高性能が 追加的なリソース消費と引き換えに得られている場合、使用できるリソース量を制限すると 最適化しないほうが高性能となる例 を示している。

## Paxos extentions

### Multi Paxos

平行稼働する Basic Paxos の複数の Proposer から リーダー を選出して、リーダーにクライアントと、アクセプターとの通信を任せ、 プロトコルにおける 最初のフェーズで プロポーザーとアクセプターが Prepare, Promise を交換する通信処理を省略する最適化方式。
端的には Basic Paxosをより軽く、よりスケーラブルにすることを目指して、安定稼働の期待できる プロポーザーをリーダーにして プロトコル の一部を省略したもの。

リーダーに処理が集中することがスループット向上に対するボトルネックとなり、課題視された。

### EPaxos (Egalitarian Paxos)

クライアントリクエストに順序付けを行うこと(オーダリング) によって一貫性を保証。(**Operation based CRDT**)  
プロポーザーとアクセプターとを同居させることで Multi Paxos をリーダーレスにする最適化。[paper](https://www.usenix.org/conference/nsdi21/presentation/tollman)

CRDT [[1](https://qiita.com/everpeace/items/bb73ec64d3e682279d26)][[2](https://www.slideshare.net/ShingoOmura/crdt-in-15-minutes-59247995)] のアイディアが反映されている。

CRDT(2011) → EPaxos(2013) → SDPaxos(2018)

CRDTはレプリカへの操作を 可換 (commutative) なものに限定して 最後にマージすると 「全てのレプリカは 操作列が全て到達すれば、収束する」という手法 例: カウンタ  
⇒ ステートマシンへのコマンド列を可換なものに限定すると、操作列が全て到達すれば全てのステートマシンレプリカは一意に収束する

### SDPaxos (Semi-Decentralized Paxos)

(EPaxosとは逆に) クライアントリクエストに順序付けを行わず、レプリカ(アクセプター)へ各コマンドを複製した後に、レプリカで全てのコマンドに順序を与えて一貫性を保証する。(**State based CRDT**) [paper](https://www.microsoft.com/en-us/research/publication/sdpaxos-building-efficient-semi-decentralized-geo-replicated-state-machines/)

### PigPaxos

ランダムなフォロワーに リーダーとの通信の 中継役 をさせることで (ピギーバックによる通信集約で) 通信コストをリーダーから(中継役の)フォロワーに分散させる方式。

<!--
![image](https://user-images.githubusercontent.com/11642047/129136665-6084bbdb-86ed-4fed-bc16-006725137930.png)
-->
