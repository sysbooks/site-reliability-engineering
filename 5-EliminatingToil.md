### Toilの定義
- toilは'やりたくない仕事'ではない
  - 人によっては手動や反復作業が好きな人もいる
  - administratice choresやGrugy work(泥臭い仕事?)も一概にはtoilとは言えない
- administrative chores
  - やらなくてはいけないadministrative choresはtoilとするのはちょっと違う。それらはoverheadと定義する
  - overheadとは直接サービスの運営には結びつかないが必要なもののこと
    - team meetings
	- setting and grading golas
	- snippets(Googlers record short free-form summaries, or “snippets,” of what we’ve worked on each wek)
	- HR papaerwork
- Grugy work
  - 長期間で価値がある場合がある。この場合はtoilではない
    - アラート設定を綺麗にしたりとか誤解されそうな箇所(複雑な箇所)を除いたりとか
- では何がtoilか
  - 下記の特性を持つもの(下記の特性を持つものがすべてtoilというわけではない)
  - manual
    - taskをまとめたスクリプトを手動で実行するようなこと
	- 一個ずつtaskをやるよりマシだが、hands-on timeがtoilだ
  - repetitive
    - 一回目や二回目のタスクはtoilではない
	- 何度も何度もやるようなことはtoilだ
	- 難しい問題や新しいソリューションを開発している場合はtoilではない
  - automatable
    - 機械が人間がやるようにできるタスクはtoilである
	- 人間が判断する必要があるタスクはtoilではない (there's a good chance)
    ```
We have to be careful about saying a task is “not toil because it needs human judgment.” 
We need to think carefully about whether the nature of the task intrinsically requires human judgment and cannot be addressed by better design. 
For example, one could build (and some have built) a service that alerts its SREs several times a day, where each alert requires a complex response involving plenty of human judgment. 
Such a service is poorly designed, with unnecessary complexity. 
The system needs to be simplified and rebuilt to either eliminate the underlying failure conditions or deal with these conditions automatically. 
Until the redesign and reimplementation are finished, and the improved service is rolled out, the work of applying human judgment to respond to each alert is definitely toil.
    ```
  - tactical
    - toilはプロアクティブで戦略的ではなくリアクティブで割り込み駆動である
	- ちょっとしたアラートを処理するのなんかはtoilである
	- このタイプのtoilは完璧には削除できないが、継続的に少なくする努力をする
  - NO enduring value
    - あなたがタスクをし終わったあとサービスがそのままであった場合はおそらくtoilである
	- もしGrugyWorkであったとしても永続的、長期的にサービスを良くするのであればtoilではない
	  - レガシーコード、コンフィグを調査し、シンプルにすることとか
  - O(n) with service growth
    - サービスのサイズに応じて線形にスケールするタスクはおそらくtoil
	- 理想を言えばサービスが拡大してもタスクが増えないこと
### なぜtoilを減らすといいのか
- toilを放置するとすぐに全員の仕事がtoilで100%になってしまう
- toilを50%に設定しているのはそのためである
- The work of reducing toil and scaling up services is the“Engineering” in Site Reliability Engineering.
- Engineeringは副次的に長期的なtoilを削除することにも繋がる
- Engineering work is what enables the SRE organization to scale up sublinearly with service size and to manage services
more efficiently than either a pure Dev team or a pure Ops team.

### toilの計算の仕方
- 6人ローテでon-callシフトが入れば2/6で33%
- 8人ローテでon-callシフトが入れば2/8で25%
- 11章にon-call シフトとか書いてある
- 緊急ではないアラート、Eメール > 緊急のアラート
- Even though our release and push processes are usually handled with a fair amount of automation, there’s still plenty of room for improvement in this area.
- toilは急激に増えたりするので、半期、四半期、一年毎のaverageでみる
- 一時的にtoilが80%とかはしょうがない

### What Qualifies as Engineering?
- 人間の判断が入る仕事
- 継続的、長期的なサービス改善に結びつきstrategyに基づくもの
- 設計ドリブンのアプローチで問題解決
- 単純に言えばより良くするということ
- Engineeringは大きなサービスを同じ人数のスタッフで支えられるようにすること
- 大体下記のような感じのカテゴリーがある
  - software engineering
    - コード直したりとか設計とかドキュメンテーションとか
	- 自動化スクリプト、fレームワークやツールの作成、スケールするサービス機能、よりロバストにするためにinfrastructure codeを修正すること
  - system engineering
    - システム構築、設定ファイルの修正、システムのドキュメンテーション、
	- モニタリングのセットアップ、アップデーととかロードバランサの設定、サーバの設定、OSパラメータのチューニング、アーキテクチャ、設計のコンサルティング(開発者に対して)
  - toil
    - Work directly tied to running a service that is repetitive, manual, etc.
  - Overhead
    - Administrative work not tied directly to running a service.
	- 採用、HR papaerwork, ミーティング、bug queue hygine, snippets, 研修とか
- Every SRE needs to spend at least 50% of their time on engineering work, when averaged over a few quarters or a year.
- 長期間toilが50%を超えたら何かがおかしいので振り返って改善する

### toilは常に悪いのか
- toilは常にみんなをunhappyにするわけではない
  - Predictable and repetitive tasks can be quite calming.
  - They produce a sense of accomplishment and quick wins.
- 少量なら問題ない、多すぎる場合が問題だ
- 多すぎるとなぜ悪いのか
  - career stagnation
    - キャリアの進みが遅くなってしまう
	- 報酬がでるがキャリアは与えられない
  - slow progress
    - 多すぎるtoilはチームの生産性をさげる
	- 手作業が多くなり機能追加ができない
  - sets precedent
    ```
If you’re too willing to take on toil, your Dev counterparts will have incentives to
load you down with even more toil, sometimes shifting operational tasks that
should rightfully be performed by Devs to SRE. Other teams may also start
expecting SREs to take on such work, which is bad for obvious reasons.
	```
  - promotes attrition
    - toilが多いとtoilが少ない他のチームを探し始める
  - causes breach of faith
    - 新入社員もやる気なくすし、契約と違う?(SREは50%engineeringで雇っている)

### 結論
- すこしずつよいengineeringをすればよい
- toilを少なくして、システムをクリーンにしてサービスサイズのスケールに対応できるようにしよう
- Let’s invent more, and toil less.

