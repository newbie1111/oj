# Design Doc

link: [DESIGN.md](https://github.com/online-judge-tools/.github/blob/master/DESIGN.md) of [online-judge-tools](https://github.com/online-judge-tools) organization


## Objective

`oj` コマンドは、競技プログラミングの問題を解くことを補助する基本的な機能を提供する。
特に、サンプルケースによるテストや自分で生成したランダムケースによるテストを行なうための機能を提供する。


## Goals

-   ユーザが個別の問題を解く速度を上げること
-   ユーザのレートを上げること
-   パソコンが苦手な人でも使える自動化の手段を提供し、競技プログラミングにおける標準的な環境のレベルを引き上げること
-   提出前のサンプルケースでのテストは必ずする、バグったら愚直解との突き合わせをして撃墜ケースを探す、という振舞いを競技プログラミング界隈における常識にすること


## Non-Goals

-   ユーザのコンテストにおける立ち回りを改善すること
    -   個別の問題を解く速度を上げることで副次的にこれも改善されるが、これをそのものを目標とはしない
-   ユーザの環境を便利にすること
    -   ユーザの行動を誘導するための手段として便利さを利用するが、これをそのものを目標とはしない


## Background

競技プログラミングにおいて、問題文中で入力の例およびそのような入力に対する正しい出力の例が与えられるのが通常である。
これらはサンプルケースと呼ばれる。
解法コード提出前にこれらサンプルケースに対する応答が正しいかを確認するのが通常である。
しかしこれは人間が手でやるには面倒であるので、省略してしまったり間違えてしまったりしやすい (例: <https://twitter.com/yamerarenaku/status/1307319917188792320>)。
これは自動化によって解決されるべきである。

`oj` コマンド (あるいはその前身の [nodchip/OnlineJudgeHelper](https://github.com/nodchip/OnlineJudgeHelper)) の登場と普及の前は、ほとんどすべての競技プログラマはサンプルケースのテストを手と目視で行うか、そもそも行なっていなかったことに注意したい。


## Overview

それぞれのオンラインジャッジとの通信は [online-judge-tools/api-client](https://github.com/online-judge-tools/api-client) に任せ、[online-judge-tools/oj](https://github.com/online-judge-tools/oj) の中では基本的に通信をしない。


## Detailed Design

-   設定ファイルは作成しない。
    -   設定ファイルの作成はユーザの環境を多様にしユーザサポートを困難にするためである。「なぜか動かない」と困っているユーザに対し必ずまず初めに「設定ファイルはどうなっていますか？」と聞かなければならない状況は避けるべきである。
-   処理はできる限り安全側に倒す。
    -   特に `test` サブコマンドにおける AC 判定のデフォルトは厳しくし「手元でサンプルが AC したならば、提出先でもサンプルには AC する」を期待できるようにする。
    -   「手元でサンプルが AC したならば、提出先でもサンプルには AC する」を健全性と呼び「手元でサンプルが WA したならば、提出先でもサンプルには WA する」を完全性と呼ぶことにしよう。RE については忘れる。健全性も完全性も持たないならば「手元での結果と提出先での結果には相関があるが、具体的に保証できる性質はない」ということになってしまうため、どちらかは成立させたい。スペシャルジャッジなどがあるので完全性は明らかに不可能である。一方で健全性は可能であるので、これはできる限り保つもとのする。
-   ディレクトリ構造については「問題ごとに異なるディレクトリを用いる」ことを仮定し、それ以外の仮定をしない。
    -   無闇に仮定を強めることは異なるディレクトリ構造を利用しているユーザに対しての導入障壁となる。一方で「自分でテストを追加する」という行動を推奨するために `test/` ディレクトリの作成は強制する。
-   役割範囲はできるだけ狭める。テンプレートコードの生成やコンテスト一括でのサンプルケースの取得機能などは [online-judge-tools/template-generator](https://github.com/online-judge-tools/template-generator) に移譲する。機能を減らしかつ単純化することで、パソコンに不慣れなユーザでも利用しやすくするためである。
    -   便利ではあるが `oj` ではサポートしない機能については他のツールで対応してもらうことになる。
    -   `oj` コマンドの目指すところは [`ag` コマンド](https://github.com/ggreer/the_silver_searcher)や [`bat` コマンド](https://github.com/sharkdp/bat)のような強力な発展的コマンドではなく [`find` コマンド](https://linux.die.net/man/1/find)や [`grep` コマンド](https://linux.die.net/man/1/grep)や [`cat` コマンド](https://linux.die.net/man/1/cat)のような安定した標準的コマンドである。

TODO: もうすこし詳しく書く


## Security Considerations

特になし。
内部で利用している [online-judge-tools/api-client](https://github.com/online-judge-tools/api-client) の問題は継承する。


## Privacy Considerations

特になし。
ユーザのデータは `submit` サブコマンドを除いて送信されない。


## Metrics Considerations

PePy <https://pepy.tech/project/online-judge-tools> を見るとユーザ数の概算が得られる。
個別の機能の利用状況についての統計は得られない。

2020/09/20 時点では、AtCoder に参加する競技プログラマの 1 割程度が利用していると言ってよいだろう。
最新版が公開されると毎回 1000 人程度がダウンロードしてくれていることが分かる。
なにかエラーがでて動かなくなるまでバージョン更新をしてくれないユーザのことを考えると 2000 から 3000 人程度のアクティブユーザを見込めるだろう。
AtCoder にはアクティブかつ茶色以上のユーザが 22000 人おりこれを潜在的ユーザの全体とする。よって概算で 1 割である。


## Testing Plan

GitHub Actions を用いて Linux, macOS, Windows すべての環境で end-to-end tests をまわす。
ファイル書き込みやコマンド実行などの入出力操作が重要となるため unit tests は特定の機能を除き役に立ちにくい。