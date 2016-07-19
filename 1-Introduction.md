# Introduction

## Introduction

- Ben Treynor SlossがSREの名付け親
- Chapter 1ではBen Treynor SlossがSREについて話す
  - 旧来のIT産業で行われていたことの違いについても

- でかくて複雑なシステムはそれ自身では動かない！
- システムはどのように動くべき？

### 旧来型のシステム管理者的なアプローチ

- 歴史的に会社は複雑なシステムを運用するためにシステム管理者を雇ってきた
- システム管理者は現在あるソフトウェアを解析してそれらをデプロイする
- システム管理者はサービスを運用し発生するイベントやソフトウェアのアップデートに対応する
- システム管理者の方法はよく知られており、真似る先もたくさんあるのでシステム管理者初心者チームでも車輪の再開発を行う必要がない
- システム管理者アプローチは関節費用と直接費用の両面で問題がある
- 直接費用は、システム管理者は手作業で事象に対応するのでトラフィックの増加等により人がたくさん必要になる
- 間接費用の増加は、分かりづらいが直接費用よりもコストがかかる部分でもある
  - devとopsの分離により、背景やソリューションに対するリスクの見方等が異なる2つのチームでの対立が発生する(ここの訳文がよくわからなかった)
  - "The split between the groups can easily become one of not just incentives, but also communication, goals, and eventually, trust and respect. This outcome is a pathology."
  - 素早くリリースする vs 安定性のためにリリースを減らす等
  - 過去に発生した全ての大障害に対応する、デプロイ時の非常にたくさんのチェックが必要とされ、するとdevチームはlauncheを減らしflag flipsを増やす方向に走る(flag flipsとは？)
  - launchレビューを減らすために少ない機能をもったサービスの分割にはしる(マイクロサービスdis？)


### Google’s Approach to Service Management:

- SREとは、ソフトウェアエンジニアにops teamのデザインをお願いした時にできるもの、という説明がシンプル
- 自分がソフトウェアエンジニアであり、自分がSREチームをデザインした
- SREの5~6割はソフトウェアエンジニアとして雇われた人、残りはソフトウェアエンジニアとしてのスキルは必要に満たずスキルは85%-99%だったけど、ソフトウェアエンジニアの持ってないUNIX system internalやnetworkingのスキルを持っている人
- 上記どっちのSREも、同じくらいいい感じに働いてる
- 多様なバックグラウンドがいい方向に働いてる
- SREは手作業は退屈だから手作業を機械に置き換えるためのソフトウェアスキルを持っている
- SREは自動化を行う
- そうしないと、サービスの成長にともなって人をたくさん雇う必要が出てくる
- opsには50%の時間を割いて残りはコードを書く
- とにかく自動化。何かが起きても勝手に復旧するような自動化
- 計測したりあらたに人を雇ったりしてops50%以下を守っている
- サービスの成長に対してsublinearlyにしか人は不要
  - 非線型？log n的な？
- SREはいろいろな課題がある
- 雇用が大変
- Googleにおけるソフトウェアエンジニアの獲得との競合
- 高いコーディングスキルと高いシステムエンジニアスキルが要求されると、雇用できる人は限られる
- アプローチがオーソドックスではないため、高いレイヤによるサポートが必要とされている。例えば、エラーバジェットが上がってしまったので四半期のリリースをやめるなどは、product development teamからは反発されるので、managementなしでは不可能


### DevOps or SRE?

- DevOpsとSREの中心となる原則は近い
- DevOpsの方がより広範な範囲を含んでいる。組織、管理機構等

### SREの信条について

- SREはops作業が50%を超えたら、ops作業を開発チームにredirectする
- それによって開発者も手作業調査が不要なシステムを作るように向けることもできる
- 休日対応的なon-call shiftで、1回のon-callでは平均的には障害2回までとしている。それ以上だと自体の収束やpostmortemの作成に十分な時間が充てられなくなるし、疲れる
- それ以上だと、疲れてしまう。逆に1回以下だと、彼らの時間を無駄にしてしまう
- 重要な問題が発生した場合はpostmotrtemsを作成する。それらは次回の事象をなくすもしくは小さくするために作る
- blame-free postmortem culture
- error budgetによってイノベーションとstabilityのコンフリクトを防ぐ
- 稼働率100%を追うのは大抵の場合無駄
- 以下を見て目標稼働率を決める
  - どのレベルの稼働率だとユーザは幸せか
  - 稼働率に満足しなかった場合に代替のサービスはあるか
  - 稼働率の違いでユーザによる製品の使い方には何がおきるか
- error budgetが決まったのであれば、大障害は悪いことではなく、innovationのためのプロセスの一つとなる
  - むしろ、error budgetを使い果たす範囲でより高速なローンチを目指したりする

### Monitoring

- モニタリングの結果をメールで飛ばすのでは弱い。人間が介在する必要が出てきてしまう
- 理想はアラートを飛ばさずに自動で復旧すること
- monitoringのoutputとしてはalerts, tickets, loggingの3つがある。

### 変更管理

- 70%の大障害は変更によって発生する
- 変更の際に、IMplementing progressive rolloutと問題の早期検知、及び変更のroll backが自動でできることによってoperationに関わる人を減らすことができる

### 需要予想とキャパシティプランニング

- capacity plannningは自然増と、イベントや機能追加などによる非自然増があり、それらに対応できるように考える
- リードタイムを考える
- 非自然の需要増を自然増の予測に加える
- システムのテストをする
- SREは、稼働率を確保するため、自然にcapacity plannningの担当となる。これはまたprovisioningおｎ担当になることも表している

### プロビジョニング

- プロビジョニングはcapacity planningの一部。サーバ購入してもprovisioningによって投入できないと意味が無い
- リソースの効率的な使用はめっちゃ大事
- サービスのスローダウンはcapacityの減少と等価
- monitorしてサービスを変更してperformanceを上げる


### The End of the Beginning

- SREは旧来型の、大きなサービスをメンテする方法とは違うよ

## Chapter 2

- Googleでの開発環境
- googleのDCは他の一般的なDCとは異なる
- このchapterではGoogleのDCを特徴づける & 紹介するこの本を通して使われる語彙について紹介する

### Hardware

- ほとんどのGOogleのコンピュータリソースはGoogleにデザインされたDC内で動いている
- 一般的なDCとは異なり、GoogleのDCは国境を超えている(?)
- 誤解をなくすため、この本では以下の用語を使う
  - Machine: hardwareの一部(もしくはVM)
  - Server: サービスを実装するソフトウェアの一部。
- MachineはどのServerをも動かすことができる。メールサーバでもBorgのサーバでも
- よく考えたら、"Server"という言葉は使わず、Machineと共に動く"binary that accepts network connection"という用語をよく使っていたが、MachineとServerという区別は重要
- 図の説明 10個のMachineがラックを構成し、複数ラックでクラスタを構成し、DCは複数のクラスタを収容している
- 物理的に近くのDC群をCampusと呼んでいる
- DC内のMachineはそれぞれに通信が必要でJupitarという名前の数万ポートのvirtual switchを作った
  - " Jupiter supports 1.3 Pbps bisection bandwidth among servers"
- DCはB4と呼ばれるglobe-spanningなバックボーンネットワークで相互接続している
  - B4はOpenFlowを使ったSDN

### System Software That “Organizes” the Hardware

- single clusterの中で1年間で数千台のMachineが死に数千台のDiskが壊れた
- それらをソフトウェアで管理するので、サービスを運用しているチームの手をわずらわせることがない
- Campus単位でhardwareをメンテナンスしたりinfraを見るチームがある

### Managing Machines

- Borgという、Apache Mesosのようなクラスタ管理システム
- Borgはユーザのjobを管理する
  - サーバやバッチのプロセスをMapReduceのように実行できる
  - ジョブは複数のtaskで成り立つ
- Borgでのjob実行は、まずtaskを実行するMachineを選定し、taskの状態をモニタリングする。taskの実行が失敗したら別のMachineで再度最初から実行する
- Machineを超えてタスク実行するので、taskの参照にIPなどは使用できない。もっと上のレベルで頑張ってる
  - Borg Naming Serviceというのを使ってtaskを参照するとのこと
  - /bns/<cluster>/<user>/<job name>/<task number> という形式での問い合わせが<IP address>:<port> に変換される
- jobへのリソース割り当ての責任もBorgは持っている
- 全jobの必要なリソースのリストをBorgsは読んで、最適な形でMachineに配分する
- taskが必要とした分よりもたくさんのリソースを使おうとした場合は殺して再起動させる

### Storage

- jobは一時記憶としてのローカルディスクを使うことができるし、永続的なstorageを使うこともできる
- LustreやHDFSのようなクラスタファイル・システムを持っている
- Dが実際にデータを保持するファイルサーバ。Colossusがreplicationや暗号化担当。どこにどのデータがあるかも管理している。ColossusがGFS(Google File System)の後継者
- Colossusの下にBigtable, Spanner,等のDBシステムが載っている

### Networking

- Googleのネットワークは複数の方法で管理されている
- OpenFlowベースなSDN
- スマートなルーティングができるという話の高価なハードの代わりに、頭の悪い高価なスイッチを組み合わせている
  - このあたり、ネットワークの知識がなくよくわからず…後の文も読むと、distributionスイッチみたいなのがなくて、central switchが頑張っている？ということ？
- Borgが、taskの使えるコンピュートリソースを制限しているように、帯域を制限している。これによって平均使用可能帯域を最大化させる
- 複数のclusterで動いているジョブを持つサービスもある。それらは世界中で動いているが、latencyを削減すうｒために、ユーザから近い場所のDCを選ぶ
- GSLBは以下の3レベルでロードバランスさせる
  - DNSを使ったジオグラフィックなLB
  - ユーザサービスレベルのロードバランス(??)
  - RPCレベルのロードバランス(? Chapter  19参照)
- サービスのオーナーはサービスのシンボリックな名前を持ち、それに複数のBNSがぶら下がっている。GSLBはBNSのアドレスに直接トラフィックを流す

### Other System Software

#### Lock Service

- ChubbyはDCを超えて使える、ファイル・システムのようなインターフェースのロック機構を提供
- 形成合意にはPaxosを使用
- Chubbyはjobのレプリカの中からmasterを決めるのにも使用されている
  - 最初にロックをとれたjobがmasterになる的な感じだろうか？
- 永続化したいデータを置くのにも便利なので、BNSはChubbyにBNSとIP:Portの組の対応をもたせている
  - Lockとどう関係があるのかよくわからない…

#### Monitoring and Alerting

- Borgmonというmonitoring programが大量に動いてモニターしている
- monitorしたいサーバのメトリクスをscrapeして
- アラート発報やグラフのためのデータ保存を行っている
  - アラート発報
  - 挙動の比較のためのデータ
  - どのリソースの消耗が続いているかのチェック

### Our Software Infrastructure

- ハードを最大限に活用するためにソフトウェアアーキテクチャが設計されている
- コードは激しくmultithreadedされているので、1つのtaskでたくさんのコアを使う
- dashboardの表示やmonitoring, debugのために全サーバは自分の診断結果やtaskの状態を表示するためのHTTPサーバを持っている
  - server-status的な
- Googleの全サービスはStubbyと呼ばれるRPC基盤を使用してやり取りしている
- Googleも"frontend/backend"(client/server)という呼び方をしている
- Apache's Thrift.のようなprotobufs(protocol buffers)(シリアライズフォーマットとのこと)を経由してRPCは転送される
  - Apache Thriftというのはfacebookが開発したRPCフレームワークとのこと
- protobufsは構造化データをシリアライズするのにXMLを使うのと比べてたくさんの利点がある
  - 圧倒的に小さく、圧倒的に早い、そして曖昧さがない

### Our Development Environment

- 開発速度は重要なので自分たちのインフラを使用して完全な開発環境を構築した
- オープンソースのリポジトリを持っている数グループ以外は、Googleのソフトウェアエンジニアは1つの共有リポジトリを使用している
- こうすることで、プロジェクトの外の問題を修正し、pull erquest的なもの(changelist or CL)を送ることができる
- エンジニア自身のプロジェクトでも、レビューが必要。単一リポジトリによって全てのソフトウェアがレビューされる

### Shakespeare: A Sample Service

- どのようにサービスが本番環境うにデプロイされるかの説明のために、あるサービスを想像してレイトする
- ある単語がシェークスピアの書いた文章に含まれるかどうかを教えてくれるサービスを作ると改定する
- そのために以下2つのサブシステムに分割する
  - シェークスピアの文書を読んでindexを作成し、Bigtableにindexを書き込むbatch
    - 1度もしくは複数回実行される
- エンドユーザからのリクエストを扱うアプリケーションフロントエンド
  - 全世界の人からアクセスがくる。常に動き続ける必要がある

- batchは3フェーズ
  - Mapがシェークスピアの文章を読んでそれぞれの単語に分割(並列実行することでより早く)
  - shuffle phaseが単語(とその出現場所の組)をソートする
  - reduceで上記を全て組み合わせたリストができる
  - それぞれの組はBigtableに格納される

#### Life of a Request

- ブラウザが shakespeare.google.com にアクセスする
- GoogleのDNSにアクセス
  - DNSは裏でGSLBにアクセス
- ブラウザはTCPの終端のreverse proxy(GFE)にアクセス
  - GFEはGoogleがカスタマイズしたApache
- GFEはどのサービスが必要とされているかを判断し、シェークスピアフロントエンドへRPCでHTMLの要求を送る
- シェークスピアfrontendは探したい単語を含むprotobufを作ってbackendに投げようとする
  - ここでBNSを使ってGSLB経由で使用可能なbackendサーバにアクセス
  - backendはBigTableにprotobufを使ってリクエスト
  - それを取得したbackendがfrontendにレスポンスを返す。そこからHTMLがユーザに返却される

- 上記が百ミリ秒足らずのうちに行われる
- たくさんの動くパーツが関与しているので、潜在的失敗ポイントはたくさんある。特にGSLBは大被害をもたらし得る
- しかしGoogleの厳格なテストと慎重なローンチ、加えてgracefulな縮退などの事前のエラー開発手法によって、信頼性のあるサービスを提供できている
- ユーザがインターネットの接続チェックを行うのにwww.google.comに接続できるかを確認するほど

#### Job and Data Organization

- backendは100QPSさばける
- ピークの予想QPSが3470QPSだったとすると、35のtaskが必要と思えるが、もう少し考えると、以下理由により実際は37task必要
  - update(デプロイのことか)の間、1taskは利用不可になる
  - 上記task update途中にmachineの死亡が起きると、もう1taskも使えなくなる
- このN+2の考えを全世界展開の、region毎のtaskの配分にも使える。
- 全世界展開の場合、それぞれのregionに必要なtaskを配備するがもしtaskが足りなくなった場合は別のregionにトラフィックを流すことによってレイテンシは高いがサービス継続は可能になるとのこと
- 規模の大きいregion内では、より回復力を高めるためにtaskを複数のclusterに置く
- 複数region分散する場合、レイテンシを下げるため及び何か会った時の回復のためにBigTableも複数regionにレプリケートする。BigTableは結果整合性をサポートするが、内容はそんなに更新されないので問題ない


