# Backend

[Back to README](/README.md)

[Back to Previous Chapter](/Chap1.md)

## 環境構築

本節では主にPythonの学習を行っていきます。
Pythonが使用できるように、`uv`を使って環境を構築していきます。

[`uv`公式](https://docs.astral.sh/uv/)

英語だけど頑張って読みましょう。

### `uv`のインストール

公式サイトの通りに従います。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

完了したら

```bash
uv --version
```

が実行できればOKです！

### Javaの環境構築

Javaの開発環境を構築するために、以下の手順で進めていきましょう。

#### 1. JDKのインストール

macOSの場合、Homebrewを使用してインストールできます：

```bash
brew install openjdk@17
```

インストール後、以下のコマンドでバージョンを確認できます：

```bash
java --version
```

#### 2. Mavenのインストール

MavenはJavaのプロジェクト管理ツールで、依存関係の管理やビルドを自動化します。

```bash
brew install maven
```

インストール後、以下のコマンドでバージョンを確認できます：

```bash
mvn --version
```

#### 3. プロジェクトの作成

Mavenを使用して新しいプロジェクトを作成する場合：

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

これにより、以下のような構造のプロジェクトが作成されます：

```
my-app/
├── pom.xml
└── src/
    ├── main/
    │   └── java/
    │       └── com/
    │           └── example/
    │               └── App.java
    └── test/
        └── java/
            └── com/
                └── example/
                    └── AppTest.java
```

#### 4. プロジェクトのビルドと実行

```bash
cd my-app
mvn package
java -cp target/my-app-1.0-SNAPSHOT.jar com.example.App
```

## Checkpoint1

Q1. `uv`を使用して、python3.12を使用したプロジェクトを作ってみましょう！

※CheckPoint4で同じ環境を使用するので、プロジェクトは残しておくことをお勧めします。

ヒント: Google検索してやり方を見てみましょう！公式ドキュメントにも色々書いてるので英語を頑張って読んだり、日本語訳したりして、読んでいきましょう！

Q2. なぜ`uv`のようなプロジェクト管理ツール・ランタイムバージョン管理ツールが必要なのでしょうか？

Q3. `pyproject.toml`に記載されている`dependencies`と`dev-dependencies`にそれぞれ含まれるライブラリの違いは何でしょうか?

Q4. `uv.lock`ファイルは`git`で管理すべきでしょうか？しないべきでしょうか？

Q5. `uv.lock`ファイルが存在せずに、`pyproject.toml`のみが存在する場合に、どのような問題が起こるでしょうか？

Q6. `uv.lock`ファイルに存在する、`hash`という項目はなぜ必要なのでしょうか？

## Python

Python言語について勉強しましょう！

<https://www.tohoho-web.com/python/>

特に、基本的な文法(`if`, `for`)と`Class`は重要なのでよく勉強してみてください。

`Class`についてよくわからない場合は以下のサイトを参考にしてみてください！

<https://lemon818.com/python-class/>

【動画】
<https://www.youtube.com/watch?v=F5guF1y7G48>

またPythonのデバッガPDBを使えるようにしましょう！

<https://qiita.com/kaitolucifer/items/dc58efebd72d72a8feb2>

上述の記事でPDBの基本的な使用方法が説明されています。

## Checkpoint2

Q1. Checkpoint1で作成したプロジェクトに、`fizzbuzz.py`というファイルを作成して、以下の仕様を満たすプログラムを書いてみてください。

仕様

①以下のコマンドで実行ができること

```bash
uv run python fizzbuzz.py <入力したい数字>
```

プログラムが実行されると答えが`print`されること

②`<入力したい数字>`の部分には1以上の長さの文字列が入る。(以下ではここで入力した文字列を`N`とする。)

③`N`の値によって`print`される答えを以下のように変える。

1. `N`が数字の場合
    1. `N`が3で割れる時`Fizz`
    2. `N`が5で割れる時`Buzz`
    3. `N`が15で割れる時`FizzBuzz`
    4. それ以外の場合`N`
2. `N`が数字ではない場合
    1. `ValueError`例外をスロー

以下は実行例

```bash
poetry run python fizzbuzz.py 4
# => 1
# => 2
# => Fizz
# => 4


poetry run python fizzbuzz.py 15
# => 1
# => 2
# => Fizz
# => 4
# => Buzz
# => Fizz
# => 7
# => 8
# => Fizz
# => Buzz
# => 11
# => Fizz
# => 13
# => 14
# => FizzBuzz


poetry run python fizzbuzz.py hoge
# =>
# Traceback (most recent call last):
#  File "<stdin>", line 1, in <module>
# ValueError("hoge is invalid input!")
```

Javaでの実装例：

```java
// FizzBuzz.java
public class FizzBuzz {
    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("Please provide a number as an argument");
            return;
        }

        try {
            int n = Integer.parseInt(args[0]);
            for (int i = 1; i <= n; i++) {
                if (i % 15 == 0) {
                    System.out.println("FizzBuzz");
                } else if (i % 3 == 0) {
                    System.out.println("Fizz");
                } else if (i % 5 == 0) {
                    System.out.println("Buzz");
                } else {
                    System.out.println(i);
                }
            }
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(args[0] + " is invalid input!");
        }
    }
}
```

実行方法：

```bash
javac FizzBuzz.java
java FizzBuzz 4
# => 1
# => 2
# => Fizz
# => 4

java FizzBuzz 15
# => 1
# => 2
# => Fizz
# => 4
# => Buzz
# => Fizz
# => 7
# => 8
# => Fizz
# => Buzz
# => 11
# => Fizz
# => 13
# => 14
# => FizzBuzz

java FizzBuzz hoge
# => Exception in thread "main" java.lang.IllegalArgumentException: hoge is invalid input!
```

Q2. pdbを使って、作成した処理で`N`の値を確認しましょう。

Q3. Classを使ったなんかいい課題を教えて下さい(help!)

Q4. is-a関係とhas-a関係の違いは？

## DB

Webアプリケーションにおけるデータを保管するデータベース(以下DB)について勉強していきましょう。

実際のWebアプリケーションではORM(Model)と呼ばれるDBを抽象化(細かい操作は見えなくして簡単に扱えるようにすること)したライブラリが使われます。

しかし、実際のサービスではORMしか知らないと、N+1問題やクエリの実行計画を見れないなどのWebアプリケーションのパフォーマンスを十分なものにすることはできません。

そこで、まず最初はDBとはどういうものか、そしてDBを操るSQLとはどういったものなのかをさらっと理解しておきましょう。

- [DB入門](http://www.isc.meiji.ac.jp/~ri03037/ICTdb1/step01.html)
- [Progate SQL入門](https://prog-8.com/courses/sql)
- [適切なIndexを張るために](https://qiita.com/kodai-saito/items/541e4fe46c2d3edc9634)

## Checkpoint3

Q1. DBにIndexを張るメリットとデメリットとは何でしょうか？

Q2. デッドロックになる場合はどのような場合でしょうか？

Q3. N+1問題とは何でしょうか？

Q4. SQL100本ノックをやりましょう(1~30番ぐらいまではやってみましょう。それ以降はお好みで)

(Google Colaboratoryで実行できます。)
<https://note.com/nmt_rootassist/n/nf70b6e73f673>

## FastAPI

いよいよWebアプリケーションのバックエンドを開発していきます！

まずは公式のチュートリアルやドキュメントを読んでみましょう！

<https://fastapi.tiangolo.com/ja/tutorial/>

## Checkpoint4

Q1. 以下の単語はどのような意味か説明してください。

1. RestAPI, エンドポイント, URI
2. HTTPリクエスト、レスポンス
3. Session
4. Cookie
5. HTTPステータスコード 例えば(404)
6. HTTPリクエストメソッド
7. CSRFトークン
8. HTTPとHTTPSの違い
9. リクエストボディとリクエストヘッダ

Q2. 以下のサイトを参考にTODOアプリのエンドポイントを作ってみてください。

環境は、Checkpoint1で作成した環境を使用してください。ライブラリ管理は`pip`ではなく、`poetry`を使用してください！

<https://dev.to/nditah/develop-a-simple-python-fastapi-todo-app-in-1-minute-8dg>

## 次のChapterを始める前に

本リポジトリを`clone`して,改善点を見つけて`Pull Request`(以下、PRと省略)を出しましょう！
どんな些細なPRでも構いません！:pray:
改善点の`Pull Request`が出されたら、本Chapterのフィードバックと次Chapterのプランを作成するための面談をセットするので改善点の`Pull Request`はどんな内容でもいいので必ず出してください！

[Go to Next Chapter](/Chap3.md)
