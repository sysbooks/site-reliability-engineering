Dealing with Interrupts
=======================

- 複雑なシステムのoperational load(運用負荷みたいな感じ？)は車の運用の用に色々と手間をかける必要がある
- システムを作った人間は完璧ではないことを忘れるな
- operational loadは最終的に"pages", "tickets", "ongoing operational activities"に分けられる
- pagesはサービスのアラートに関心があり、単純でよく起きてなんの思考もなしに対処できるものもあるが深い思考が必要なpagesもある
  - しばしば"○分"で表現されるSLOを持つ
- ticketsはアクションが必要な顧客からの要求
  - pagesと同様に何も考えずに行う退屈なもの(configについてのコードレビューとか)もあればreal thoughtが必要なもの(unusualなcapacity planとか)もある
  - ticketsもSLOがあるが、"○時間"や"○週間"というレベル
- ongoing operational responsibilitiesは(also known as “Kicking the can down the road” and “toil”; see Chapter 5)
  - team-owned code or flag rollouts, or responses to ad hoc, time-sensitive questions from customers
  - これらはSLOを持たず、あなたをinterruptし得る
- operational loadの中には予想して計画に組み入れることができるものもあるが大半はそうではなく、よくない時に誰かをinterruptし得る

## Managing Operational Load

Googleはチームレベルに応じてoperational loadの複数の管理法を持っている

### Pages

- on call engineerは大抵1人でpagesに対処している
- on call engineerはユーザサポートや開発チームへのエスカレーション等もしている
- pageがチームに与えるinterrupts及び傍観者効果を減らすために1人のエンジニアが割り当てられる
- pageの内容が不明だった場合は別のチームにエスカレートする
- secondary on call engineerもいて役割は様々だが、ただpageがfall throughしてきた場合にprimaryに連絡をとるだけの場合もある

### Tickets

- ticketsの管理方法はチームによっていくつかある
- primaryやsecondaryなon call engineerが対応したりチーム内にtickets personがいたり
- ticketsはランダムにチームメンバーに分配されたりチームメンバーが取っていったりする

### Ongoing operational responsibilities 

- これもtickets同様、様々な方法で管理されている
- on call engineerが行ったりチームメンバーにアサインされたり、数週間かかるチケットの場合はon call engineerが自分のシフト週に実施するようにpick upする

## Factors in Determining How Interrupts Are Handled

- interruptsをどう管理するかについては以下factorを考えると良い
- InterruptのSLOや期待する反応時間
- 本来backlogに記録されるべきinterruptsの数
- interruptsの深刻さ
- interruptsの頻度
- 特定の種類のinterruptに何人が対応可能か

## Imperfect Machines

- 人間はimperfect

### Cognitive Flow State

- flow stateはエンジニアによく知られている概念。
- "zone"は生産性を向上させる。芸術的科学的creativityも
- interruptsはが十分に混乱をもたらすレベルだった場合flow stateはすぐにいなくなってしまう
- この状態の時間を最大化させることが重要
- Cognitive Flowはよりクリエイティブではないもの(家事やドライブ)に対しても適用され、flowのための重要な要素は満たされている
  - 明確なゴール、即座のフィードバック、時間の歪み
- low skillで難易度の低い問題によってzoneを手に入れることができる
- 同様に、エンジニアが直面するようなhigh skillで高難易度の問題でも同様にzoneに入ることができる
- flow stateに至るまでの方法は異なるが得られる利益は一緒

### Cognitive flow state: Creative and engaged

- 問題に取り組むのが心地よく、解ける！と思い本気で問題に取り組むinterruptsを出来る限り無視する
- この状態を最大化することが重要。クリエイティブな結果を出し、量もこなし、やっている仕事ができて幸せだと感じる
- 不幸なことにSREのようなロールはこの状態になろうとしては失敗しフラストレーションが溜まるもしくはこの状態になろうとしなくなる

### Cognitive  ow state: Angry Birds

- どのように行えばいいかを知っているタスクを行うことを人々は好む。実際、そのようなタスクを実行することはcognitiveに到達するための明確な道である
- cognitive flowになるのがon-call時のSREもいる
- 逆にコード書いたり別のプロジェクトをしている際にon-callになったりfull timeのinterruptにさらされると大変なストレスになる
- 一方、常にinterruptにさらされている人にとってはinterruptがなくなることがinterruptである
- incrementalな改善を行うこと、巨大なチケット、問題や障害の修正は明確なゴール、境界、フィードバックがある
- あなたがinterruptをするなら、あなたのプロジェクトは気が散るものになる
- project/on-callの2つの状態になっても最終的にはその2つをうまくバランスさせることに幸せを感じる
- エンジニアによってその理想的なバランスは異なるが、重要なのは自分にとってのいいバランスを知らないエンジニアがいるということ

## Do One Thing Well

- ここで話すのはGoogleのSREチームに何が効果的だったかに基づいた提案であり、チームマネージャーやinfluencerへの利益を主に考えている
- ここではチームの個々人の習慣は考えない(個々人で考えてやってくれ)これによってチームがinterruptsをどのように管理するかのチームの構造について考えることができる

### Distractibility

- cognitive flowを妨げられることはたくさんある
- 例えばSREだけど今日はon-callじゃないFredが"Do Not Disturbヘッドフォン"をして仕事をし始めました。これでZone Time？実際のところは以下のように色々起きる。結果として彼はカレンダー上ではFreeでプロジェクトに集中できるはずだったのに気が散りまくる状態に
  - Fredのチームはチケットの自動ランダムアサイン機能を使っていて、チケットがFredにあたりました
  - Fredの同僚がon-callでFredがよく知ってるコンポーネント周りについてのpageを受けとりFredに話しかけてきた
  - Fredのサービスのユーザが先週Fredがon-callの際にFredに割り当てられたticketの優先度を上げてきた
  - Fredが担当していたflagのrolloutが誤っていてroll backした
  - Fredはhelpfullな男だからFredのサービスのユーザが質問をしに来た
- 上記のいくつかはIMをオフにするとかで自身で対応可能
- ポリシーやinterruptsとongoing responsibilitiesへの仮定によって起きるdistractionもある
- 避けられないdistractionもあるが、チームとしてinterruptを管理して平均的にundistractibleにすることが可能

### Polarizing time

- コンテキストスイッチを減らそう
- コンテキストスイッチにコストを与える。1つのプロジェクトwork中に20分のinterruptを入れるとコンテキストスイッチは2回になり真に集中する2時間が消し飛ぶ
- 正常的な時間の消滅を防ぐため、work style間の時間の分離を行う。具体的には、プロジェクトに取り組む時間とひたすらinterruptに取り組む時間を分ける。理想的には一週間毎だが、現実的には1日毎や半日毎

## Seriously, Tell Me What to Do

- 断片的に実行できる様々なプラクティスをここに提示するよ！

### General suggestions

- interruptが多すぎた場合は人を増やせ
- ticketもそうだしon-call時のpageも同様。pageをsecondaryに投げたりpageをticketにdowngradeするなど

### On-call

- primary on-callエンジニアはon-callの仕事だけに集中すべき
- もしpagerが静かなら、すぐに片付けられるticketやinterupt baseな仕事をするべき
- その週がon-callの当番である場合、その週はプロジェクトの仕事は0にすべき。プロジェクトが重要なのであればそのエンジニアはon-callにはすべきではない
- secondary on-callエンジニアはどのような義務を課せられているかによってプロジェクトにアサインできるかが決まる
- (あなたのチケットが0になることはありますがからクリーンアップ作業がなくなることはありません。ドキュメントのupdateやconfigのclean upや将来に役立ちます) なんでここにこれが書かれているかわからなかった

### Tickets

- チケットのアサインをランダムにやっている場合はやめろ
- primary/secondary on-callエンジニアで処理できる以上の量のticketがある場合は、ticket処理の専任担当を決めて処理すべき。チーム全体に分散させるべきではない

### Ongoing responsibilities

- 可能な限り、マントを受け取るロールを定める
- ある変更を行う手順が明確に決まっているのであれば、その変更をずっと追いかける(その変更で何かあったら行った人がそれに対処する)必要はない
- on-callやon-interruptsの間にpushesをやりくりするpush managerを定義すべき
- 自分が行ったことの後始末を他人が行うことはコストがかかるが、on-callじゃないエンジニアがuninterruptedな時間を作るための軽いコストである

### Be on interrupts, or don’t be

- on-interruptsではないエンジニアが行う価値があるinterruptなタスクをチームが受け取る場合があるが、それはできるだけ減らすべき
- 人々は自分に割り当てられていないチケットを始める場合があるbecause it’s an easy way to look busy。これはよくない
- 1人の人間にアサインしているのに複数人が実施している場合、気付かず管理不能チケットキューを持っていることになる

## Reducing Interrupts

- 任意の時点であまりにも多い人数がinterruptsに対応している場合はあなたのチームの割り込み負荷は管理不能であるといえる
- 減らす方法がある

### Actually analyze tickets

- 多くのチケットのローテーション及びon-callのローテーションは籠手のように機能する。特に大きいチームにおいて
- もしあなたが2ヶ月ずっとinterrupts対応をしていたら棒打ちの系を受けるのは簡単。後継者も同様のことをして、チケットの根っこの原因は解明されない。前進するよりもむしろ同じissueにいらつき行き詰まる
- on-call workと同じくらい、ticketのためにhandoffをすべき
  - handoff processはチケットハンドラ間の共有状態を維持する
- 割り込みの要因の基本的な振り返りは割り込み率の低下のための良い解決策を与える
- 主原因を発見できるか確認するために割り込みの種類を検査する中であなたのチームはチケットやpageの棚卸しをすべき
- 主要因が妥当な時間で修正できると考えた場合は、主要因が修正されると期待できる時間までまでinterruptsをだまらせることでinterruptsを扱う人を安心させ、主要因を修正する人に簡単な締め切りを作ることとなる

### Respect yourself, as well as your customers


