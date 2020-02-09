# Zettelkastenメソッド

Zettlrを作るアイデアが浮かんだのは、数年前にアカデミックな文書を書くワークフローを改善しようとしていた時のことです。様々なスタイルやワークフローのアイデアをテストし、その中から選ばれたのがZettelkastenメソッドでした。その当時は、Zettelkastenをうまく実装したソフトウェアはほとんどありませんでしたが、それから多くの実装の努力が続けられ、現在ではZettelkastenの機能をサポートするアプリケーションが増えてきています。

もともと、このメソッドはドイツの社会学者ニクラス・ルーマン(Niklas Luhmann)によって生み出されました。彼は、自分が読んだり考えたことをすべて思い出せるように、情報と番号を書いたカードを使った(当時はアナログでした)Zettelkastenをデザインしました。番号は、カードの情報に関連した情報が書かれた他のカードを探すために使えました。ルーマンはこの方法でカード間の参照を行き来しました。そして、箱がカードでいっぱいになってくると、これが何らかの形で活きて、自分で考えもしなかったような思考のつながりが見えてきました。

つまり、小さなメモどうしに（または長いファイルでも構いませんが）関連付けを行うことで、ファイル間を行ったり来たりするだけでなく、ファイル間に浮かび上がる関連性を明らかにするというのが、基本的なアイデアです。

## ZettlrでZettelkastenを管理する

Zettelkastenを始めるために、Zettlrでは今のところ次の3つの機能が利用できます。

1. ファイルのIDを生成する
2. 検索とファイルへのリンクを作る
3. ファイルにタグを付ける

### ファイルID

ファイルにアプリケーション固有の情報を残さないようなアプリケーションを作ろうとすると、個々のファイルの識別方法が大きな問題になります。Zettlrは、内部的にファイルを区別するために32bitのハッシュ値を使っています。しかしこれはファイルのパスに依存した値です。Zettlrはクラウドに保存されたファイルも扱えるようにデザインされていて、別のコンピュータ上ではパスが同一とは限らないことから、このハッシュ値をIDとして使うことはできません。もう一つの問題は、フォーマット自体にあります。Markdownファイルはプレーンテキストであるため、余分なメタデータを加えることができません。もちろん、Markdownファイルにヘッダーを追加するというアプローチもあるのですが、その方法はとれませんでした。なぜなら、アプリケーションに依存しないMarkdownファイルを作るというZettlrの原則を破ってしまうからです。Markdowの文法に比べると、メタデータはあまり標準化されていないので、これをZettlrの哲学と両立させる方法は簡単には思いつきません。そこで、私たちは単にIDを文章中に埋め込むことにしました。ファイルにIDを追加するには`Cmd/Ctrl+L`を押してIDを生成します。もしくは、手動でIDを入力すれば、自動的にIDがハイライトされます。

> [設定のページ](../reference/settings.md)で、リンクに対するZettelkastenの設定項目をご確認ください。

デフォルトのIDは良い選択肢です。なぜならYYYYMMDDHHMMSSの日時を使っているので、毎秒ごとに一意な値になるからです。また、経験的に言えることですが、簡単に認識できないIDの方がファイルを相互にリンクするのに向いています。自分で試してみてください。

Zettlrは自動的にファイルの内容を検索してIDを見つけます。発見したIDがWikiリンク形式(詳細は後述)で囲まれていない場合、それをファイルのIDとみなします。

### 内部リンク

IDの問題が解決すると次の問題が発生します。前述のように、ファイルをアプリケーションに依存させないというZettlrの目標を達成しつつ、どうすればファイル間にリンクを張ることができるでしょうか。nvALTやThe Archiveのような多くのアプリケーションでは、ファイルの相互参照によりシステム内をできる限り簡単にナビゲートできるような、内部リンクシステムを実装しています。Zettlrも同様に、そのようなシステムを持っています。

内部リンクは`[[This is the link]]`のような書式で書かれます。`Alt`もしくは`Ctrl`を押しながらリンクをクリックすると、**2つの**異なる機能が実行されます。一つ目は、アプリケーション中でリンクの内容に完全一致するものを探します。つまりリンクに完全一致するような内容を持ったファイルを検索します。この完全一致には2つのパターンがあります。一つはリンクの内容（上の例だと「This is the link」）がファイル名の拡張子を除く部分に完全に一致する場合です。上の例の場合、`This is the link.md`、`This is the link.markdown`、`This is the link.txt`などが完全一致と判定されます。ファイル名の比較は大文字小文字が無視されることにご注意ください。例えばmacOSなどではデフォルトで大文字小文字が無視されます。（なので`filename.md`と`FILENAME.MD`は同一のファイルを示します。）もう一つは、リンクの中身が`[[<your-id>]]`のような書式でIDを含んでいる場合です。この場合、`<your-id>`というIDを含んでいるファイルが、完全一致であると判定されます。**Altを押しながらリンクをクリックして、完全一致のファイルがシステム上で見つかった場合、最初に見つかったファイルが開かれます。**つまり、このようなリンクを使用して、システム中をナビゲートすることができます。例えば、いくつかのファイルへの内部リンクを含む目次ファイルを作り、それぞれのファイルには目次ファイルへの逆リンクを張るような応用が可能です。

リンクから実行される二つ目の機能は、選択中ディレクトリに対するグローバル検索です。これは単に、リンクの中身を検索欄に貼り付け、Enterを押して検索を開始します。このようにして完全一致したファイルを開くだけでなく、そのファイルにリンクした他のファイルを見つけることができます。`[[<your-id>]]`という書式のリンクがあった場合、IDに対応するファイルが開かれるとともに、そのファイルにリンクしているすべてのファイルが検索されます。

### タグ付け

タグ付けは内部検索の中では最も簡単かもしれません。`Alt`または`Ctrl`を押しながらタグをクリックすると、現在のディレクトリでそのタグが付けられたすべてのファイルが検索されます。タグは`#keyword`のような形式ですが、これはMarkdownの他の書式では使われていません。このようなタグを使ったアプローチにより、Zettlrでは完璧なタグ付けシステムを構築できています。