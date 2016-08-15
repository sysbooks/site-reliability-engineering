9 Simplicity
============

(全体的に意訳しまくったメモ。)

ソフトウェアはdynamicでunstableなものである。
'At the end of the day, our job is to keep agility and stability in balance in the system.'

## System Stability Versus Agility

- SREの仕事は、ソフトウェアのreliabilityをより高めるようなprocedures, practices, toolsを作ること
- ただし、developerのagilityをできるだけ損なわないように
- 実際にはdeveloperのagilityが損なわれるようなことはあまりなく、reliableなprocessはむしろdeveloperのagilityを向上させる。
  - reliableなprocessにより、プロダクションへのデプロイが素早くなり、バグがみつかっても、そのバグの修正に時間がかからなくなる

## The Virtue of Boring

- ソフトウェアにとって、予想がついて意外性がなく'boring'であることは、ポジティブな意味をもつ。
  - "Surprises in production are the nemeses(天罰の女神) of SRE."
- essential(本質的) complexityとaccidental(偶発的) complexityの違いは重要
  - Webサーバを書くことは高速にウェブページを返すというessential complexityを伴う。
  - しかし、WebサーバをJavaで書いたとして、GCのパフォーマンスインパクトを最小化しようとするのはaccidental complexity
- accidental complexityをなるべき最小化しましょう。

## I Won’t Give Up My Code!

- エンジニアは自分のコードを消したがらない
  - あとで必要になるかも。単にコメントアウトしておけばあとで使えるかも。消すよりもフラグで分岐するようにするか
- 24 hours/7 weeksの可用性を期待するなら、すべての新しいコード行は負債である。
  - SREは、すべてのコードにessential purposeを持つことを可能にするpracticeを奨励する。例えば、business goalをdriveするものかを調査したり、dead codeを雑に削除したり、bloat detection(肥大化検知の仕組み？)をtestingのすべての段階に組み込んだり。

## The “Negative Lines of Code” Metric

- software bloat(ソフトウェアの肥大化)は直感的には望ましくないが、SREの観点から考えたときに、その負の側面はよりはっきりする。
- すべてのコードは、新しい欠陥やバグを引き出すポテンシャルをもつ。
- we should perhaps entertain reservations when we have the urge to add new features to a project (entertain reservationsがよくわからなかった)

## Minimal APIs

- サン＝テグジュペリ「完璧が達成されるのは、何も加えるものがなくなった時ではなく、何も削るものがなくなった時である。」
  - これはソフトウェアにも当てはまる。APIは特に。
- clearでminimalなAPIはソフトウェアにおけるsimplicityを管理するための重要な要素。

## Modularity

- Googleの「protocol buffers」

## Release Simplicity

- Simple releases > complicated releases
- リリースが小さいバッチで構成されていれば、コードの各変更は独立していて理解できるので、我々はよりはやく自信をもって動ける。
- このアプローチは機械学習における勾配降下法と似ている。小さなステップをとり、各段階の結果が良かったかどうかをみて最適解を見つける。

## A Simple Conclusion

- software simplicity is a prerequisite(前提条件) to reliability


