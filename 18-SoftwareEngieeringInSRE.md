Software Engineering in SRE
===========================

## Why Is Software Engineering Within SRE Important?

- サードパーティツールではGoogleのスケールに適さないことが多いため、内部ソフトウェア開発が必要
- SREは内部ソフトウェアを効率的に開発するためのユニークなポジションである。
  - scalabilityやgraceful degradation during failure、他のインフラストラクチャやツールとのインタフェースなどの要素に対して適切に考慮されたソフトウェアを設計し、作成できる
  - SREsはsubject matterなので、開発されたツールの必要性や要求を簡単に理解できる
  - 開発者(intented user)と直接やりとりできる
- SREチームの人数はサービス成長に対してスケールするべきではない。チームをスケールさせるには、自動化などが必要。
- 本格的なソフトウェア開発プロジェクトはSREsのキャリアに開発の機会を与える。長期プロジェクトになると、割り込みとon-call workに対するバランスが必要で、software engineering と systems engineering のバランスを維持したいエンジニアにとってよい
- SREにはチームの多様性が必要で、Googleは常にSREチームを、伝統的なソフトウェア開発経験とシステムエンジニアリング経験をもつエンジニアを混ぜる。

## Auxon Case Study: Project Background and Problem Space

- Auxon: a powerful tool developed within SRE to automate capacity planning for services running in Google production.
- 計算機リソースのキャパシティプランニングには無数の戦術がある (https://www.usenix.org/publications/login/feb15/capacity-planning) しかし、これらのアプローチの大部分は以下のcycleに落とし込める。
  - 1. Collect demand forecasts. どれくらいのリソースが必要か、いつ、どこでそれらのリソースが必要か。
  - 2. Devise build and allocation plans. 要求を満たすのにどの方法がベストか。 How much supply, and in what locations?
  - 3. Review and sign o  on plan. forecastがreasonableかどうか。 with budgetary, product-level, and technical considerations?
  - 4. Deploy and configure resources.

### Brittle by nature (本質的に脆い)

- 伝統的なキャパシティプランニングにより、どんな表面上のマイナーチェンジにも崩壊しうる計画が作られる。例えば、
  - serviceの効率が下がる。より多くのリソースが必要になる
  - 新クラスタのdelivery dateが遅れる
  - パフォーマンス目標に関するプロダクトの意思決定が、要求されたservice deploymentと要求リソース量、の形を変化させる
- あるクオータのためのキャパシティプランは、前回のクオータのキャパシティプランから期待される結果にもとづいている。それはつまり、あるクオータが変化すると、それ以降のクオータを更新する結果になる。

### Laborious and imprecise (面倒で不正確)

- すべてのforecast(予測)には制限とパラメータがあり、制限はintentと関連する
  - レイテンシ要求を考えた場合、North Americaの余剰リソースは、Asiaでは役に立たない
- 制限されたリソース要求を、利用可能なキャパシティから実際のリソース割り当てにマッピングすることは、等しく遅い。
- ツールで解決しようとしても大抵、unreliable or cumbersomeになる。スプレッドシートはスケーラビリティに問題やerror-checkingが制限されている問題がある。
- 人間の手でビンパッキングをやるのは厳しい。
- 最終的に、ビンパッキングをする人間のリソースを大量に消費する。このプロセスは変更するほどもろくなり、最適解には程遠い。

(おそらく我々がやってるような、将来のリソース消費量を眺めつつ、この物理サーバは、この用途に割り当てて、みたいなやり方は厳しいということが書いてある)

### Our Solution: Intent-Based Capacity Planning

- Specify the requirements, not the implementation.
- Googleの多くのチームでは、Intent-based Capacity Planningのアプローチに移行している。
  - このアプローチの基本は、サービス要求のdependenciesとparameters (intent)をプログラムでencodeするようにすること。
  - リソースがどのサービス、どのクラスタに割り当てるかの詳細なプランを自動生成する
  - なにかが変化しても、変化したパラメータを与えてやれば新しいプランを自動生成できる
- ビンパッキング問題をコンピュータに委譲することにより、human toilを減らせる
- より精確で、コスト削減につながる。ビンパッキングは解決された問題ではないが、今日のアルゴリズムなら十分に最適。

## Intent-Based Capacity Planning

- intentはサービスオーナーがサービスをどのように運用するかについて根拠がある
  - intent-based capacity planningは複数のレイヤの抽象を要求する
  - 1. I want 50 cores in clusters X, Y, and Z for service Foo.
  - 2. I want a 50-core footprint in any 3 clusters in geographic region YYY for service Foo.
  - 3. I want to meet service Foo’s demand in each geographic region, and have N + 2 redundancy.
  - 4. I want to run service Foo at 5 nines (99.999%) of reliability 
- 理想的にはすべてのレベルのintentを同時にサポートすべき
- Googleの経験では、ステップ3. がintentとimplementationのバランスがよい

### Precursors to Intent

### Introduction to Auxon

### Requirements and Implementation: Successes and Lessons Learned

### Raising Awareness and Driving Adoption

### Team Dynamics

## Fostering Software Engineering in SRE

### Successfully Building a Software Engineering Culture in SRE: Sta ng and Development Time

### Getting There

## Conclutions

- ソフトウェアエンジニアリングプロジェクトは組織が成長するにつれて、もりあがる。

