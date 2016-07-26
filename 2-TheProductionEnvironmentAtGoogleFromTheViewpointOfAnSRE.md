## The Production Environment at Google, from the Viewpoint of an SRE

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



