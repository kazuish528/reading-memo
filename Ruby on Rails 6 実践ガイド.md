|||
|:---|:---|
| Title | Ruby on Rails 6 実践ガイド |
| ISBN | 978-4295008057 |
| Errata | https://book.impress.co.jp/books/1118101134 |

----

改修作業などで特に触れていない部分の知識を補完する感じに読む。

## Chapter 1〜2

環境構築なのでskip。Dockerで説明されていたので、自分でRailsプロジェクトを新規作成する時に参考にできるか？

## Chapter 3

初期設定。

### Gemfileに追加されていたGem

* bcrypt
  * パスワード暗号化用 ……この本ではdeviseを用いずに認証を実装するからか
* rails-i18n
  * 翻訳ファイル
* kaminari
  * ページネーションサポート
* date_validator
  * 日付バリデーション
* valid_email2
  * メールアドレスバリデーション
* nokogiri
  * XMLパース

### 設定

`config/database.yml` にデータベースの接続設定、`config/application.rb` にタイムゾーンやデフォルトロケールの設定を入れる。

#### i18n.load_path

```rb
config.i18n.load_path += Dir[Rails.root.join("config", "locales", "**", "*.{rb,yml}").to_s]
```

`config/application.rb` に入れるよう指示されているこれが何なのかと思って調べたが、ロケールファイルをディレクトリ管理できるようにするためのものだった。  
見たことあるプロジェクトだと `"#{locale}.yml"` くらいしか置かれていなかったが、確かに内容が多くなってくるとディレクトリ分けられたほうが嬉しいのかも。


#### config/initializers/blocked_hosts.rb

Rails 6からの機能。リリースノートには「DNSリバインディング攻撃からの保護」と書かれている。Same Origin Policyを破れるのか…  
デフォルトではdevelopment環境で有効。

参考: https://railsguides.jp/6_0_release_notes.html#railties-%E4%B8%BB%E3%81%AA%E5%A4%89%E6%9B%B4, https://techracho.bpsinc.jp/hachi8833/2020_02_05/83154

## Chapter 4

RSpecの使い方について。

### 保留

```patch
 example "foo" do
+  pending
 end
```

or 

```patch
-example "foo" do
+xexample "foo" do
```

## Chapter 5

ルーティングの初歩的な設定、ERBの使い方、ヘルパーメソッドの定義方法。あとアセットパイプライン。

### アセットパイプライン

設定は `app/assets/stylesheets/application.css` 内のディレクティブや `config/initializers/assets.rb`
 を見る。

`rails assets:precompile` でプリコンパイル。

### credentials.yml.enc

Rails 5.2からの機能。`secret_key_base` などを暗号化して管理するための仕組み。`secrets.yml` は廃止された。

この辺が参考になりそう: https://qiita.com/15grmr/items/a687d0ed211ef60e751c

#### 概要

* `credentials.yml.enc` には秘匿情報を暗号化して書き込める
* 内容を編集するには `rails credentials:edit`
* 暗号化された値を取り出すには `Rails.application.credentials.xxx`
* 復号には `config/master.key` ファイルか `ENV['RAILS_MASTER_KEY']` の設定が必要
* `config/master.key` はバージョン管理してはならないとされている
  * しかし、開発環境では共有しないと困るのでは...?
  * `config/master.key` 自体は本番しか考慮されてなさそう。

もうちょっと読み込んで、Rails 5.1以下からバージョンアップしているアプリで何かする必要はあるのか覚えておいたほうが良さそう。

## Chapter 6

エラー処理。

### config.exceptions_app

ルーティング例外など、`rescue_from` で拾えないもののハンドリングに使える。

```rb
Rails.application.configure do
  config.exceptions_app = ->(env) do
    # 例外処理
    # Controller.action.callすればエラーページを起動するとかできる
  end
end
```

## Chapter 7

マイグレーション、モデルの作成、セッションの利用方法。  
特に目新しいことは無かった。

## Chapter 8

(Viewにおける) フォームの作成方法と、フォームオブジェクト・サービスオブジェクトの説明。

### フォームオブジェクト

フォームを作成する際に、ActiveRecord Modelに直接依存せずに実装できるようにするパターン。  
ActiveRecord Modelと同様の機能を使えるように `ActiveModel::Model` をincludeするのがミソ。

Railsアプリケーションでは `app/forms` に置かれることが多い模様。

```rb
class FooForm
  include ActiveModel::Model

  # attr_accessor を書いていく
end
```

### サービスオブジェクト

コントローラにベタ書きせずに、ビジネスロジックと呼ばれるような類のものを独立実装できるようにするパターン。

これ自体は特に真新しいものではない……。  
Railsアプリケーションでは `app/services` に置かれることが多い模様。

Fat controller → Fat model → Service object の流れがあったとするコラムは少しためになった。  
過去にいた現場ではあまりFat model化の流れは来ていなくて、その手前のFat controllerはかなり見た。自分たちの首が着実に絞まっていくのであれは辛い。

## Chapter 9

ルーティング。だいたい他のRailsフォロワーとできることは同じ。

RSpecでルーティングが適切に動作しているか確認したい時は、マッチャーとして `route_to` や `be_routable` が使える。

## Chapter 10

ActiveRecordのCRUD。この辺はRails Guidesに書いてあることを説明しているだけ。

Rails Guidesにも書いてあるけど見落していたこととして、 `#new_record?` を呼ぶと、レシーバが新規に作成されてまだ永続化される前のものか確認できる。

`save`, `save!` や `destroy`, `destroy!` の動きの違いは分かるが、それをどう使い分けるかという例を見てみたい気持ち。

## Chapter 11

### Strong Parameters

`params` を生で使うと、雑に `assign_attributes` なんかに渡した日には任意の入力を受け入れてしまうので、必要なものだけを抽出して扱うようにする。

常に `#[]` で一つずつ取り出しているからあまり恩恵が無いという場所もありそうだが、`params#require` で必須を付けるのは常に恩恵があるので、習慣として使っておくのが良さそう。

Laravelではバリデーターがこの手の入力フィルタの役目も持っていたのを思い出した。バリデーションの立ち位置の違いはあるが。  
一方、近い構造をしているであろうCakePHPはこういうのが存在していたか忘れた。今ならありそう。

### attributes_for

FactoryBotの機能のひとつで `create(:factory_name)` に近い。違いとしては、こちらはレコードは作らずにファクトリーで定義した値の入ったHashを返す。

### be_* マッチャー

本書の例でいうと `be_suspended` のような形でマッチャーを指定すると、ターゲットオブジェクトの `#suspended?` を呼んだ結果でテストできる。

## Chapter 12

アクセス制御と書かれているが、内容としては `before_action` の使い方。

## Chapter 13

DBの外部キーの説明と、ActiveRecordの関連付け。

### dependent: オプション

本では一部に触れていたが、元々よく覚えていなかったのでRails Guidesを写経して覚える。

#### has_one, has_many の場合

| 値 | 効果 |
|:--|:----|
| :destroy | 関連付けたレコードのdestroyを実行 |
| :delete | 関連付けたレコードを直接DBでDELETE |
| :nullify | 外部キーをNULLにする |
| :restrict_with_exception | 関連付けられたレコードが存在する場合、ActiveRecord::DeleteRestrictionError例外 |
| :restrict_with_error | 関連付けられたレコードが存在する場合、`#errors` にその旨を追加 |

#### belongs_to の場合

| 値 | 効果 |
|:--|:----|
| :destroy | 関連付けたレコードのdestroyを実行 |
| :delete | 関連付けたレコードを直接DBでDELETE |

## Chapter 14

バリデーションと、`before_validation` を利用した正規化の紹介。

サンプル、メールアドレスが入力トリム以上に正規化されているのはどうかと思う。

## Chapter 15

### Presenter

あるオブジェクトに関連するビュー周りのコードを集めておくレイヤーを作る考え方。  
これをパッケージングしたGemとしてはDraperやActiveDecoratorがある。

自分で実装した場所で、けっこうControllerで無理した場所があったと思うので、こうすれば良かったのかという感想。

## Chapter 16

単一テーブル継承 (STI) の説明。

同じテーブルを使いながら、typeカラムに識別用の値を入れることでさもオブジェクトの継承のようにバリエーションを持たせられるようにする。  
Railsにおいては、識別用の値には具象クラス名が用いられる。

だいぶRDBの正規化の主義からは離れている気が。

抽象クラス → 保存先テーブル として紐付けされるものの、実際にはそのクラスを継承した具象クラスで操作する形になる。

```sql
create table base (id integer primary key, type varchar(255) not null);
```

```rb
class Base < ApplicationRecord; end
class ImplA < Base; end
class ImplB < Base; end

# 操作にはImpl{A,B}を使う。共通する部分はBaseに実装して、独自の部分はImpl{A,B}に…といった、OOPの継承そのままの考えで実装する。
# baseのテーブルに全てのattributeが網羅されるようにマイグレーションを書かないと、当然動作しなくなる。
```

いわゆる区分値を持たせているようなテーブルなら、結果的に適用できることもあるのかもしれない。  
それだったら読みやすくなることもありそう。  
ただ、簡単にRDBの思想から踏み外せるのであまり濫用したくはない気持ち。

## Chapter 17

Capybara を用いたE2Eテストの説明。  
実務だとほとんどSPAになってしまっているが、それでも使えるものなのだろうか？

SPAの場合、フロントエンド側のエコシステムからE2Eテストフレームワークを採用しそうなので、今回はskip

## Chapter 18

ここまでのサンプルコードに仕様を付け加える演習の章なのでskip

## 雑感

Railsフォロワーのフレームワークをいくつか触っていたのもあって、真新しい内容は少なかった。概念そのものからというより、Railsでの作法を覚えることのほうが多い。

だいたいは既に業務で触っているアプリケーションでは実践されていることだったりして、ここから活かせるものを探すよりも理由を知らずに触っていた部分の答え合わせが主となった。

新しめのRailsで導入された機能については、まだ業務で導入されてない部分もありそうなので、そこを掘り下げておくと後々活かせそう。

後はもっとステップアップした内容の本があればと思うが、ちょっと検索した程度だと全然出てこなかったのでどうしたものかと悩んでいる。

## どこか使えそうか？

* Chapter 3のgemリストはいつでも参考にできそう。
* Chapter 5の `credentials.yml.enc` は別の資料で勉強したい。  
  `secrets.yml` が完全廃止されることは当分無いとは思うが、移行方法を把握しておきたい。
