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

サービスのintentを知るために必要な情報は何か？ dependencies、performance metrics、prioritization。

Dependencies

(マイクロサービス的な各サービスの依存関係の話。プロダクション環境では依存関係がネストしているとか)

Performance Metrics

- dependenciesの連鎖を理解することは、ビンパッキング問題の定式化に役に立つ。しかし、リソース使用量についてもっと情報が必要だ。
- サービスFooがN user queriesを処理するのに必要な計算機リソースはどれくらいか。サービスFooのすべてのN queriesに対して、サービスBarのデータ転送量は何Mbpsか。
- Performance Metricsはdependencies間の糊。より高いレベルのリソースタイプからより低いレベルのリソースタイプに変換できる。

Prioritization

- リソースの制約は、結果としてトレードオフになる。すべてのサービスの要求の中から、キャパシティ不足に直面してどの要求を犠牲にするか。
- サービスFooのN+2の冗長性は、サービスBarのN+1の冗長性よりも重要。もしくは、Xの機能リリースは、サービスBazのN+0の冗長性よりも重要ではない。
- Intentベースのプランニングは、これらの決定を強制的に、透過的にオープンに一貫性があるようにする。

### Introduction to Auxon

- Auxonは、Intentベースのキャパシティプランニングとリソース割り当てソリューションの、Googleの実装である。
  - SREが設計・開発したソフトウェアエンジニアリングプロダクトの最上の例
  - ソフトウェアエンジニアの小さなグループとテクニカルプログラムマネージャが2年かけて作った
- Auxonは、サービスのresource requirementsとdependenciesに関する記述を収集するための手段を提供する。
  - Requirements: "My service must be N + 2 per continent" or "The frontend servers must be no more than 50 ms away from the backend servers."
  - a user configuration language or a programmatic APIを介して、この情報を収集する。human intent => machine-parseable constraints
  - requirementsは内部的に巨大な混合整数線形計画法で表現されている
- Figure 18-1  The major components of Auxon
- `Performance Data` describes how a service scales
  - for every unit of demand X in clus‐ ter Y, how many units of dependency Z are used? 
  - いくつかのサービスは負荷テスト、その他は過去のパフォーマンスからの推量
- `Per-Service Demand Forecast Data` describes the usage trend for forecasted demand signals. 
  - いくつかのサービスは、demand forecastsから将来のusageを引き出す - リージョンごとのQPSの予報
  - すべてのサービスがdemand forecastsをもつわけではなく、自分に依存するサービスから純粋にdemandを引き出す (例えばストレージサービスなど)
- `Resource Supply` provides data about the availability of base-level, fundamental resources
  - 例えば、将来のある時点で利用可能なマシンの個数。
  - 線型計画法の用語では、upper bound 
- `Resource Pricing` provides data about how much base-level, fundamental resources cost
  - 線形計画法の用語では、最小化したいobjective
- `Intent Config` is the key to how intent-based information is fed to Auxon.
  - サービス構成要素とサービス間の関連
  - human-readbale
- `Auxon Conguration Language Engine` acts based upon the information it receives from the Intent Config.
  - human-configurable intent definitionとthe machine-parseable optimization requestのゲートウェイ
- `Auxon Solver` is the brain of the tool.
  - the Configuration Language Engineから受け取った最適化リクエストをベースに線計画法を定式化する。
  - スケーラブルに設計されている。数百から数千台のマシン上で並列に動作する
  - スケジューリングやワーカープールと決定木の管理など
- `Allocation Plan` is the output of the Auxon Solver.
  - どのサービスのどの場所にリソースが割り当てるべきかが書かれている

### Requirements and Implementation: Successes and Lessons Learned

- Auxonの開発を通して、SREチームはプロダクションワールドに関わり続けた。
- いくつかのGoogleサービスの on-call rorationに入り、設計の議論や技術的なリーダーシップに参加した。
- プロダクションワールドに足を付けた。consumerとdeveloperの両方として活動した。
- 機能リクエストは、チームの直接の経験を通して伝えられた。
- プロダクトのオーナーシップ感覚をもつだけでなく、SRE内で、プロダクトの信頼性や正当性を得た

Approximation

- 完璧であることや純粋な解決にこだわってはいけない。特に問題の境界がよく知られていないときは。Launch and iterate.
- 線形計画法の世界はチームメンバーにとって未知の領域だったため、最初は単純化したsolver engineを実装することを選んだ (Stupid Solver)
  - ユーザが指定したrequirementsにもとづいてサービスがどのように調整されるかに対して、いくつかのヒューリスティックを加えた
- Auxon内でsolverのインタフェースは抽象化されているため、solverの入れ替えは可能。最終的には入れ替えた。
- Auxonはfuzzyなプロダクト要求を抱えていて、fuzzyなものを作るのはフラストレーションのたまるチャレンジである。しかし、fuzzyなことはソフトウェアが汎用的でモジュール化されて設計されるインセンティブになる
  - 例えば、Auxonではautomation tool, forecasting tool, or performance data toolを入れ替えることなしに、Auxonを使い始めることができたので、"agnostic"アプローチは新しいユーザの導入のためのキーになった。
- 完璧なデザインを待ってはいけない。むしろ、ビジョンを意識せよ。

### Raising Awareness and Driving Adoption

- あなたのソフトウェアプロダクトの意識と関心を高める努力をおこたってはいけない
  - 一回のプレゼンテーションやメールアナウンスだけでは不十分
- 巨大なオーディエンスに対する内部ソフトウェアツールのSocializingは以下のすべてを必要とする
  - 一貫性のあるアプローチ
  - ユーザの主張
  - あたながプロダクトの有用性をデモンストレートするシニアエンジニアやマネージャの支援
- エンジニアは、ツールの使い方を把握するためにソースコードを掘ったりする時間はない。内部ユーザであるSREもまた忙しいので、ドキュメントは必要であり、もしあなたのソリューションが難しすぎるなら、SREは自分でソリューションを書くだろう

Set expectations

- もしプロダクトが十分に報われる成果を保証されていないなら、チームに新しいことにトライすることを納得させるために必要なエネルギーをしぼりだすことはむずかしくなりうる
- 小さくリリースして、インクリメンタルな進捗により、便利なソフトウェアを作れるチームの能力に対してユーザの信頼を得られる

Identify appropriate customers

- 多くの大きなチームは、キャパシティプランニングのための自前のツールを持っていた。完璧ではなかったが、これらのチームはキャパシティプランニングのプロセスにおいて、新しいツールを試すための痛みを経験していなかった。
- Auxonは、最初はキャパシティプランニングプロセスの存在しないチームをターゲットにした。徐々に広めていった。

Customer service

- どんな革新的なソフトウェアであっても、学習曲線はある。アーリーアダプターに対するカスタマーサポートを恐れてはいけない。

Designing at the right level

- 非依存性がAuxonにとっての key principle
- 1つや2つの大きなカスタマーのためのオーバカスタマイズを避けることにより、組織を超えたbroader adoptionを得て、新サービスの参入の敷居を下げた

### Team Dynamics

- SREソフトウェア開発プロダクトのメンバー選定において、新トピックに対して高速にキャッチアップできるGeneralistsと幅広い知識と経験をもつエンジニアを合わせた初期チームを作るメリットを発見した
  - 多様性により、blind spotをカバー
- ほとんど会社のSREチームであれば、新しい問題空間のために、タスクをアウトソースするかコンサルタントと働く。しかし、より大きな組織のSREチームはin-house expertsとパートナーになるかもしれない
  - Auxonの設計フェーズでは、デザインドキュメントをGoogleのOperations ResearchとQuantitative Analysisに特化したin-houseチームにプレゼンした
- プロジェクトが進むと、統計学や数学的な最適化のバックグラウンドをもつメンバーをチームは獲得する
- スペシャリストがジョインする正しいタイミングはプロジェクトにより異なる。ざっくりしたガイドラインを示すなら、プロジェクトはうまく発足すべきで、その結果、現在のチームのスキルがエキスパートのジョインにより支えられる。

## Fostering Software Engineering in SRE

- プロジェクトが、オンオフツールから本格的なソフトウェアエンジニアリングの努力へシフトするための、google candidateは何か？
  - プロジェクトはSREのtoilの削減や既存のインフラ要素の改善のような顕著なメリットを提供すべき
- プロジェクトが組織の目的にフィットするかは重要なことだ
  - その結果、エンジニアリーダーは直接やりとりするかもしれないチームといっしょに、プロジェクトのポテンシャルインパクトや主張を評価できる
- プロジェクトをpoor candidateにするのは何か？
  - software that touches many moving parts at once, or software design that requires an all-or-nothing approach that prevents iterative development. 
  - SREチームはサービス単位なため、一部の組織にしかメリットのない仕事をするリスクがある。プロジェクトはたびたび汎用化に失敗する
  - 逆に、汎用フレームワークが問題が多いときがある。柔軟すぎてユースケースにフィットしないなど。
- 幅広いユースケースに対応した例: a layer-3 load balancer  https://research.google.com/pubs/pub44824.html

### Successfully Building a Software Engineering Culture in SRE: Staffing and Development Time

- SREsはしばしばgeneralistである。深さ優先より幅優先で学習
  - strong coding and software development skillをもつ一方で、プロダクトチームの一員となることやカスタマーからの機能リクエストなどの、traditional SWE experienceをもたないかもしれない
  - 伝統的なSREチームアプローチ "I have a design doc; why do we need requirements?"
  - user-facing software developmentの経験者が、team software development cultureをつくるのを助けられる
- 割り込みのない project work timeは、software development effortにとって必要不可欠
- プロダクトが正式化したあとに、とりうる選択肢がいくつかある
  - エンジニアのspare timeで開発
  - structured processesを通して、正式なプロジェクトとして確立する
  - Gain executive sponsorship 
- しかし、いずれのシナリオにおいても、SRE組織に組み込まれたfull-time developersになる代わりに、SREsとして働き続けることが必要不可欠。
- プロダクションの世界に浸透することで、SRESは、creatorとcustomerの両方の視点という、かけがえのない観点を得られる。
-  Immersion in the world of production gives SREs performing development work an invaluable perspective, as they are both the creator and the customer for any product.

### Getting There

- どのようにしてソフトウェア開発モデルをSRE組織に導入するか？
  - technical challengeではなくorganizational change 
  - SREsは閉じたチームで、問題に対して高速に解析し反応する。したがって、目の前の必要性のために高速にコードを書く、SREの本能に逆らっている。
  - SREチームが小さければ問題はないかもしれない。しかし、組織が大きくなると、アドホックアプローチではスケールしない。
- 次に、SREでソフトウェアを開発して達成したいことは何かを考える
  - want to foster better software development practices?
  - are you interested in software development that produces results that can be used across teams, possibly as a standard for the organization?
  - 大きな組織では、後者の変化には時間がかかる。
- 以下はGoogleのガイドラインである

- Create and communicate a clear message
- Evaluate your organization's capabilities
- Launch and iterate
- Don't lower your standards


## Conclutions

- ソフトウェアエンジニアリングプロジェクトは組織が成長するにつれてもりあがる。
- SREドリブンなソフトウェアプロジェクトは会社にとって顕著なメリットがある
- SREsはしばしば、不十分なプロセスの流線化や、共通タスクの自動化するためのソフトウェアを開発するので、チームサイズが、サービスのサイズに線形にスケールする必要はない
- 究極的には、会社、SRE組織、SRE自身が、ソフトウェア開発に捧げる時間をSREsがもつことのメリットを享受できる。

