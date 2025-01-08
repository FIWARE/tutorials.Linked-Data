[![FIWARE Banner](https://fiware.github.io/tutorials.Linked-Data/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

<!-- prettier-ignore -->

このチュートリアルでは、リンクト・データ (Linked Data) の概念を FIWARE プラットフォームに導入します。スーパーマーケット
・チェーンのストア・ファインダ・アプリケーションは **NGSI-LD** を使用して再作成されます。**NGSI v2** と **NGSI-LD**
インターフェースの違いが強調され、説明されています。このチュートリアルは最初のチュートリアルと直接類似していますが、
**NGSI-LD** インターフェースからの API 呼び出しを使用します。

チュートリアルでは [cUrl](https://ec.haxx.se/) コマンドを使用していますが、
[Postman のドキュメント](https://fiware.github.io/tutorials.Linked-Data/) としても入手できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/125db8d3a1ea3dab8e3f)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-v2)

:warning:  **注:** このチュートリアルは、システムを **NGSI-LD** に切り替えたりアップグレードしたりする **NGSI-v2**
開発者向けに設計されています。リンクト・データシステムを最初から構築する場合、または **NGSI-v2** にまだ慣れていない場合は、
[NGSI-LD 開発者向けチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)のドキュメントを参照することをお勧めします。

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [FIWARE データ・エンティティへのリンクト・データの概念の追加](#adding-linked-data-concepts-to-fiware-data-entities)
    -   [リンクト・データとは何ですか？](#what-is-linked-data)
        -   [:arrow_forward: ビデオ : リンクト・データとは何ですか？](#arrow_forward-video-what-is-linked-data)
        -   [:arrow_forward: ビデオ : JSON-LD とは何ですか？](#arrow_forward-video-what-is-json-ld)
    -   [NGSI-LD とは何ですか？](#what-is-ngsi-ld)
        -   [NGSI v2 データ・モデル](#ngsi-v2-data-model)
        -   [NGSI v2 データ・モデル](#ngsi-ld-data-model)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [WSL](#wsl)
-   [アーキテクチャ](#architecture)
-   [起動](#start-up)
-   [リンクト・データをベースとした "Powered by FIWARE" アプリの作成](#creating-a-powered-by-fiware-app-based-on-linked-data)
    -   [サービス状態の確認](#checking-the-service-health)
    -   [コンテキスト・データの作成](#creating-context-data)
        -   [コア・コンテキスト](#core-context)
        -   [Smart Data Models](#smart-data-models)
        -   [NGSI-LD エンティティ定義内のプロパティの定義](#defining-properties-within-the-ngsi-ld-entity-definition)
        -   [NGSI-LD エンティティ定義内の Properties-of-Properties の定義](#defining-properties-of-properties-within-the-ngsi-ld-entity-definition)
    -   [コンテキスト・データのクエリ](#querying-context-data)
        -   [FQN タイプによるエンティティ・データの取得](#obtain-entity-data-by-fqn-type)
        -   [ID によるエンティティ・データの取得](#obtain-entity-data-by-id)
        -   [タイプによるエンティティ・データ取得](#obtain-entity-data-by-type)
        -   [属性値を比較してコンテキスト・データをフィルタ](#filter-context-data-by-comparing-the-values-of-an-attribute)
        -   [配列内の属性値を比較してコンテキスト・データをフィルタ](#filter-context-data-by-comparing-the-values-of-an-attribute-in-an-array)
        -   [サブ属性の値を比較してコンテキスト・データをフィルタ](#filter-context-data-by-comparing-the-values-of-a-sub-attribute)
        -   [メタデータをクエリすることでコンテキスト・データをフィルタ](#filter-context-data-by-querying-metadata)
        -   [geo:json 属性の値を比較してコンテキスト・データをフィルタ](#filter-context-data-by-comparing-the-values-of-a-geojson-attribute)
-   [次のステップ](#next-steps)
    - [License](#license)


</details>

<a name="adding-linked-data-concepts-to-fiware-data-entities"></a>

# FIWARE データ・エンティティへのリンクト・データの概念の追加

> “Six degrees of separation doesn't mean that everyone is linked to everyone else in just six steps. It means that a
> very small number of people are linked to everyone else in a few steps, and the rest of us are linked to the world
> through those special few.”
>
> ― Malcolm Gladwell, The Tipping Point

FIWARE 入門の[チュートリアル](https://github.com/FIWARE/tutorials.Getting-Started)の紹介では、コンテキスト・データ・
エンティティの作成と操作に一般的に使用される [NGSI v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
インターフェイスを紹介しました。このインターフェースの進化は、**リンクト・データ**の概念を追加しながらコンテキスト・
データ・エンティティを強化するためのメカニズムとして
[NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
と呼ばれる補足仕様を作成しました。このチュートリアルでは、新しいインターフェイスの背景にあるアイデアを紹介し、
リンクされたデータとしてデータ・エンティティを作成および操作する方法を比較対照します。

このシリーズのその他のチュートリアルでは、データのリレーションシップ、およびリンクト・データを使用してコンテキスト・
データ・エンティティを作成し、完全なナレッジ・グラフをトラバースする方法についてさらに説明します。

<a name="what-is-linked-data"></a>

## リンクト・データとは何ですか？

インターネットのすべてのユーザはハイパーテキスト・リンクの概念に精通しているでしょう。これは、ある Web ページ上の
リンクが既知の場所から別のページをロードするようにブラウザを導くことができる方法です。

人間はリレーションシップの発見の可能性とリンクがどのように機能するかを理解することができますが、コンピュータはこれを
はるかに困難に感じ、別の場所に保持されているデータ要素間を移動できるように明確なプロトコルを必要とします。

コンピュータ用の読み取り可能なリンクのシステムを作成するには、データ自体からプログラム的にセマンティックな意味を取得
できるように明確に定義されたデータ・フォーマット ([JSON-LD](http://json-ld.org/)) を使用し、両方のデータ・エンティティ
とエンティティ間のリレーションシップに一意の IDs
([URL また はURN](https://stackoverflow.com/questions/4913343/what-is-the-difference-between-uri-url-and-urn))
を割り当てる必要がありますが必要です。

正しく定義されたリンクト・データを使用してビッグデータの質問に答えることができます。また、_"Store X の棚で現在
入手可能な製品とその価格は何ですか？"_ などの質問にデータのリレーションシップをたどることができます。

<a name="arrow_forward-video-what-is-linked-data"></a>

### :arrow_forward: ビデオ : リンクト・データとは何ですか？

[![](https://fiware.github.io/tutorials.Step-by-Step/img/video-logo.png)](https://www.youtube.com/watch?v=4x_xzT5eF5Q "Introduction")

リンクト・データの概念を説明したビデオを見るには、上の画像をクリックしてください。

JSON-LD は JSON を拡張したもので、リンクト・データを JSON で表現するときのあいまいさを避けるための標準的な方法であり、
データはマシンによって解析可能な形式で構造化されます。これは、各属性が何を意味するのかについて異なる考えを持つ可能性
がある、多数の別々のデータ・ソースから取得されたときにすべてのデータ属性を簡単に比較できるようにする方法です。たとえば、
2つのデータ・エンティティが `name` 属性を持っているとき、コンピュータは同じ意味で (**Username** や **Surname**、
何かではなく) _"Name of a thing"_ を参照することが確実にできます。URL とデータ・モデルは、属性が短い形式 (`name`
のような) と完全に指定された長い形式 (`http://schema.org/name` のような) の両方を持つことを可能にすることによって
あいまいさを取り除くために使われます。

JSON-LD は `@context` 要素の概念を導入します。これはコンピュータが残りのデータをより明快かつ深く解釈できるようにする
追加情報を提供します。

さらに JSON-LD 仕様では、明確に定義された
[データ・モデル](https://smartdatamodels.org/)
をデータ自体に関連付ける独自の `@type` を定義できます。

<a name="arrow_forward-video-what-is-json-ld"></a>

### :arrow_forward: ビデオ : JSON-LD とは何ですか？

[![](https://fiware.github.io/tutorials.Step-by-Step/img/video-logo.png)](https://www.youtube.com/watch?v=vioCbTo3C-4 "JSON-LD")

JSON-LD の背後にある基本概念を説明したビデオを見るには、上の画像をクリックしてください。

<a name="what-is-ngsi-ld"></a>

## NGSI-LD とは何ですか？

**NGSI-LD** は、リンクト・データ (エンティティのリレーションシップ)、プロパティ・グラフ、およびセマンティクス (JSON-LD
が提供する機能を利用) のサポートを改善するために修正された **NGSI v2** 情報モデルの進化です。この作業は ETSI ISG CIM
イニシアチブの下で行われており、更新された仕様は
[NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf)
としてブランド設定されています。NGSI-LD の主な構成要素は、_Entity_, _Property_ および _Relationship_ です。NGSI-LD
エンティティ (インスタンス) は、プロパティまたはリレーションシップの対象になります。伝統的な NGSI v2 データ・モデルに
関しては、プロパティは属性とその値の組み合わせと見なすことができます。リレーションシップは、リンクト・データを
使用してインスタンス間の関連付けを確立することを可能にします。

<a name="ngsi-v2-data-model"></a>

### NGSI v2 データ・モデル

ちなみに、NGSI v2 データ・モデルは非常に単純です。以下のようにまとめることができます :

![](https://fiware.github.io/tutorials.Linked-Data/img/ngsi-v2.png)

NGSI v2 の中心的な要素はデータ _entity_ (エンティティ) で、通常は、**Store**, **Shelf** などの状態が変化する実際の
オブジェクトです。エンティティは `name` や `location` などの _attributes_ (属性) を持ち、これらは順番に `precision`
などの _metadata_ (メタデータ)、つまり `location` の読み取りの正確さを保持します。

すべての _entity_ はエンティティが記述する種類のものを定義する `type` を持たなければなりませんが、NGSI v2
エンティティに `type=Store` を与えることは、誰もが自分自身の **Store** エンティティを同じ方法で形作ることを余儀なく
されるように比較的意味がありません。同様に `name` という名前の属性を追加しても、他の誰かの `name`
属性と同じデータを突然保持することはありません。

NGSI v2 を使用してリレーションシップを定義することができますが、それは属性に慣例で定義された適切な属性名 (例えば
`refManagedBy` のように `ref` で始まるもの) を与え、属性 `type=Relationship` を割り当てます。
これも純粋な命名規則で、実際の意味的な重みはありません。

<a name="ngsi-ld-data-model"></a>

### NGSI LD データ・モデル

NGSI LD データ・モデルはより複雑で、ナビゲート可能なナレッジ・グラフにつながる、より厳密な使用定義があります。

![](https://fiware.github.io/tutorials.Linked-Data/img/ngsi-ld.png)


繰り返しになりますが、_entity_ はコア要素と見なすことができます。すべてのエンティティは、URI (多くの場合
[URN](https://en.wikipedia.org/wiki/Uniform_resource_name)) でなければならない一意の `id` を使用する必要が
あります。保持されているデータの構造を定義するために使用される `type` もあります。これも URI でなければなりません。
この URI は、Web 上にある明確なデータ・モデルに対応している必要があります。例えば、URI の
`https://uri.fiware.org/ns/dataModels#Building` は
[Building](https://github.com/smart-data-models/dataModel.Building) のための共通の
データ・モデルを定義するために使われます。

_Entities_ は、_properties_ と _relationships_ を持つことができます。理想的には、それぞれの _property_ の名前は、
ウェブ全体で見いだされる共通の概念に対応する明確に定義された URI であるべきです (例えば、`http://schema.org/address`
はアイテムの物理アドレスのための共通の URI です)。_property_ はそのプロパティの状態を反映する値も持ちます (例えば
`name="Checkpoint Markt"`)。最後に、プロパティはそれ自身がプロパティに関するさらなる情報を反映するさらなるプロパティ
(別名 _properties-of-properties_) を持つことができます。プロパティとリレーションシップは、リンクされた埋め込みプロパティ
(_properties-of-properties_ または _properties-of-relationships_ または _relationships-of-properties_ または
_relationships-of-relationships_ など) を持つことができます :

NGSI LD データ・エンティティ (スーパーマーケットなど) :

-   一意でなければならない `id` を持っています。例えば `urn:ngsi-ld:Building:store001`
-   明確に定義されたデータ・モデルの完全修飾 URI であるべきである `type` を持ちます。例えば
    `https://uri.fiware.org/ns/dataModels#Building`。また、作成者は、JSON-LD @context を介して完全修飾
    URIs にマップされた型の短い文字列としてタイプ名を使用できます
-   エンティティの _property_ を持ちます。例えば、ストアのアドレスを保持する `address` 属性です。これは
    `http://schema.org/address` に展開することができ、これは完全修飾名
    ([FQN](https://en.wikipedia.org/wiki/Fully_qualified_name)) として知られています
-   他の _property_ と同様に、`address` は _property_ の `address` に対応する _value_ を持ちます。例えば
    _Bornholmer Straß e 65, 10439 Prenzlauer Berg, Berlin_
-   エンティティの _property-of-a-property_ を持ちます。例えば `address` の `Verified` フィールドです
-   エンティティの _relationship_ を持ちます。例えば、`managedBy` というリレーションシップが他のデータ・エンティティに
    対応する `managedBy` フィールド: `urn:ngsi-ld:Person:bob-the-manager`
-   リレーションシップ `managedBy` はそれ自身 _property-of-a-relationship_ (例えば `since`) を持つことができ、これは
    Bod がストアの作業を開始した日付を保持します
-   リレーションシップ `managedBy` は、それ自身が _relationship-of-a-relationship_ (例えば ` subordinateTo`)
    を持つことができ、これは階層のボブの上のエリア・マネージャの URN を保持します

ご覧のとおり、ナレッジ・グラフは明確に定義されており、無制限に拡張できます。

リレーションシップについては、以降のチュートリアルで詳しく説明します。

<a name="prerequisites"></a>

# 前提条件

<a name="docker"></a>

## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、
さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Windows に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の指示に従ってください
-   Mac に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の指示に従ってください
-   Linux に Docker をインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAMLファイル](https://raw.githubusercontent.com/Fiware/tutorials.Linked-Data/master/docker-compose/orion-ld.yml)
を使用して、アプリケーションに必要なサービスを設定します。これは、すべてのコンテナ・サービスを単一のコマンドで起動
できることを意味します。Docker Compose は、Docker for Windows および Docker for Mac の一部としてデフォルトでインストール
されますが、Linux ユーザは[こちら](https://docs.docker.com/compose/install/)にある手順に従う必要があります。

## WSL

簡単な bash スクリプトを使ってサービスを開始します。Windows ユーザは、Windows 上の Linux ディストリビューションに
似たコマンドライン機能を提供するために [を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install) をダウンロードするべきです。

<a name="architecture"></a>

# アーキテクチャ

デモ・アプリケーションは、準拠している Context Broker と NGSI-LD の呼び出しを送受信します。NGSI v2 と NGSI-LD
の両方のインターフェースが [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
の実験版で利用可能であるため、私たちのデモ・アプリケーションは1つ FIWARE コンポーネントのみを利用します。

現在、Orion Context Broker は、保持しているコンテキスト・データの永続性を保つためにオープンソースの
[MongoDB](https://www.mongodb.com/) テクノロジに依存しています。したがって、アーキテクチャは2つの要素から
構成されます :

-   [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使ってリクエストを受け取る [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース :
    -   データ・エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・データ情報を保持するために
        Orion Context Broker によって使用されます

2つの要素間のやり取りはすべて HTTP リクエストによって開始されるため、要素をコンテナ化して公開ポート
から実行することができます。

![](https://fiware.github.io/tutorials.Linked-Data/img/architecture.png)

必要な設定情報は関連する `orion-ld.yml` ファイルの services セクションにあります :

```yaml
orion:
    image: quay.io/fiware/orion-ld
    hostname: orion
    container_name: fiware-orion
    depends_on:
        - mongo-db
    networks:
        - default
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion:1026/version || exit 1
```

```yaml
mongo-db:
    image: mongo:4.2
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - "27017"
    ports:
        - "27017:27017"
    networks:
        - default
    command: --nojournal
```

両方のコンテナは同じネットワーク上に存在します -  Orion Context Broker はポート `1026` をリッスンし、MongoDB
はデフォルトのポート `27017` をリッスンしています。どちらのコンテナも同じポートを外部に公開しています -
これは純粋にチュートリアル・アクセスのためのものです - したがって、cUrl または Postman は同じネットワークの一部
でなくてもそれらにアクセスできます。コマンドラインの初期化は一目瞭然です。

入門チュートリアルとの唯一の注目すべき違いは、必要なイメージ名が現在 `fiware/orion-ld` であるということです。

<a name="start-up"></a>

# 起動

すべてのサービスは、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Linked-Data/blob/NGSI-v2/services) Bash スクリプトを実行して、
コマンドラインから初期化できます。以下のようにコマンドを実行して、リポジトリのクローンを作成して必要なイメージを
作成してください :


```bash
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
git checkout NGSI-v2

./services orion|scorpio|stellio
```

> **注 :** クリーンアップして最初からやり直す場合は、次のコマンドで実行できます :
>
> ```
> ./services stop
> ```

---

<a name="creating-a-powered-by-fiware-app-based-on-linked-data"></a>

# リンクト・データをベースとした "Powered by FIWARE" アプリの作成

このチュートリアルでは、最初の _"Powered by FIWARE"_ スーパーマーケット検索アプリと同じデータ・エンティティ
を再作成しますが、NGSI v2 ではなく NGSI-LD リンクト・データ・エンティティを使用します。

<a name="checking-the-service-health"></a>

## サービス状態の確認

いつものように、公開されたポートに HTTP リクエストをすることで Orion Context Broker が実行されているかどうかを
確認できます :

#### 1️⃣ リクエスト :

```console
curl -X GET \
  'http://localhost:1026/version'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "orion": {
        "version": "1.15.0-next",
        "uptime": "0 d, 3 h, 1 m, 51 s",
        "git_hash": "af440c6e316075266094c2a5f3f4e4f8e3bb0668",
        "compile_time": "Tue Jul 16 15:46:18 UTC 2019",
        "compiled_by": "root",
        "compiled_in": "51b4d802385a",
        "release_date": "Tue Jul 16 15:46:18 UTC 2019",
        "doc": "https://fiware-orion.readthedocs.org/en/master/"
    }
}
```

バージョンのレスポンスの形式は変更されていません。`release_date` は、以下に定義されているリクエストを処理できるよう
にするために、2019年7月16日以降でなければなりません。

<a name="creating-context-data"></a>

## コンテキスト・データの作成

リンクト・データ・エンティティを作成するときは、一般的なデータ・モデルを使用することが重要です。これにより、複数のソース
からのデータを簡単に結合し、異なるソースからのデータを比較するときのあいまいさを取り除くことができます。

各属性は URI である必要があるため、全体で完全修飾名を使用してリンクト・データを作成するのは困難です。
そこで JSON-LD はコンテキスト定義へのポインタを保持できる `@context` 属性のアイデアを導入しました。
Smart Data [Building](https://github.com/smart-data-models/dataModel.Building) データ・エンティティを追加するには、次の
`@context` が必要です。

```json
{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    ...  other data attributes
    "@context": "https://fiware.github.io/data-models/context.jsonld"
}
```

<a name="core-context"></a>

### コア・コンテキスト

[https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld)
は NGSI-LD のコア `@context` を参照します。これはすべての NGSI エンティティに共通の `id` や `type` のような用語を定義し、
`Property` や `Relationship` のような用語を定義します。コア・コンテキストは NGSI-LD にとって非常に基本的なもので、
デフォルトでリクエストに送信された `@context` に追加されます。

<a name="smart-data-model"></a>

### Smart Data Models

[https://schema.lab.fiware.org/ld/context](https://schema.lab.fiware.org/ld/context)
は、FIWARE が提供する標準データ・モデルの定義を指します。これを `@context` に追加すると、GSMA や TM Forum のような他の組織と共同で、
FIWARE Foundation によって定義されたすべての[データモデル](https://smartdatamodels.org/)の定義が読み込まれます。
**Building** に関連する FQNs の概要は以下のようになります :

```json
{
    "@context": {
        "Building": "https://uri.fiware.org/ns/dataModels#Building",
        ... etc
        "address": "http://schema.org/address",
        "category": "https://smart-data-models.github.io/data-models/terms.jsonld#/definitions/category",
        "location": "https://uri.etsi.org/ngsi-ld/location",
        "name": "https://uri.etsi.org/ngsi-ld/name",
        ...etc
    }
}
```

このコンテキスト定義を含めると、エンティティの `Building`, `address`, `location`に短い名前を使用できるようになりますが、
他の情報源と比較するときにコンピュータも FQNs を読み取ることができるようになります。

#### Context terms are IRIs not URLs

[JSON-LD仕様](https://www.w3.org/TR/json-ld/#the-context) によると : _"コンテキストは Terms を IRIs にマップするために
使用される" ことに注意してください。_ - IRI (Internationalized Resource Identifier) は必ずしも URL ではありません -
[ここ](https://fusion.cs.uni-jena.de/fusion/blog/2016/11/18/iri-uri-url-urn-and-their-differences/)を参照してください。
したがって、`https://uri.etsi.org/ngsi-ld/name` などの要素が実際に Web ページに解決されなくても、予期しないことでは
ありません。ただし、`http://schema.org/address` などの JSON-LD `@context` ファイル内の多くの IRIs は、実際に、
それ自体に関する詳細情報を含む Web ページを返します。

NGSI-LD [Core @context](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld) を使用する場合、

```json
{
  "@context": {
    "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
    "geojson": "https://purl.org/geojson/vocab#",
    "id": "@id",
    "type": "@type",
...
    "@vocab": "https://uri.etsi.org/ngsi-ld/default-context/"
  }
}
```

属性の未解決のショートネームがデフォルトのコンテキストにマッピングされることがわかります:

-   不明な属性 `xxx` => `https://uri.etsi.org/ngsi-ld/default-context/xxx`

そして当然のことながら、これらのデフォルト・コンテキストの IRI は有効な Web ページとしても存在しません。

Context Broker に有効な **Building** データ・エンティティを作成するには、以下に示すように
`http://localhost:1026/ngsi-ld/v1/entities` エンドポイントに POST リクエストを行います。データ・エンティティが Linked data
として認識されるように、適切な `Content-Type: application/ld+json` も使われることが重要です。

#### 2️⃣ リクエスト :

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": "commercial"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
             "coordinates": [13.3986, 52.5547]
        }
    },
    "name": {
        "type": "Property",
        "value": "Bösebrücke Einkauf"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}'
```

Context Broker は `@context` で言及されているすべてのファイルをナビゲートしてロードしなければならないので、
最初のリクエストは時間がかかります。

> **注**: 何らかの理由でリクエストが失敗して、`https://schema.lab.fiware.org/ld/context` が利用できない場合、
> FIWARE データモデル `@context` の代替コピーは、GitHub の FIWARE データモデル の gh-pages にあります。
> @context 値を以下でで置き換えます:
> `https://fiware.github.io/data-models/full-context.jsonld`.
>
> 稼働中の本番システムでは、サードパーティがコンテキストを確実に読み取ることができるように、`@context` ファイルが常に
> 利用可能であることが不可欠です。このチュートリアルでは、アーキテクチャをシンプルに保つために、高可用性
> インフラストラクチャを考慮していません。

#### 3️⃣ リクエスト :

後続の各エンティティは、与えられた `type` に対して一意の `id` を持たなければなりません

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities/ \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store002",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": "commercial"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
     "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
              "coordinates": [13.3903, 52.5075]
        }
    },
    "name": {
        "type": "Property",
        "value": "Checkpoint Markt"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}'
```

<a name="defining-properties-within-the-ngsi-ld-entity-definition"></a>

### NGSI-LD エンティティ定義内のプロパティの定義

`id` と `type` 属性は NGSI v2 を使ったことのある人なら誰でも知っているはずで、これらは変わっていません。上記のように、
タイプはインクルードされたデータ・モデルを参照するべきです、この場合 `Building` はインクルードされた
URN `https://uri.fiware.org/ns/dataModels#Building` の短い名前として使われています。その後、各 _property_ は2つの属性、
`type` と `value` を含む JSON 要素として定義されます。

_property_ 属性の `type` は以下のいずれかでなければなりません :

-   `"GeoProperty"`: ロケーション については`"http://uri.etsi.org/ngsi-ld/GeoProperty"`。
    [GeoJSON 形式](https://tools.ietf.org/html/rfc7946)で経度と緯度のペアとして指定する必要があります。
   プライマリ・ロケーション属性の推奨名は `location` です
-   `"Property"`: `"http://uri.etsi.org/ngsi-ld/Property"` - 他のすべてのものについては。
-   時間ベースの値の場合、`"Property"` も使用されますが、プロパティ値は、
    [ISO 8601形式](https://en.wikipedia.org/wiki/ISO_8601) にエンコードされた Date, Time または DateTime
    文字列でなければなりません。例えば `YYYY-MM-DDThh:mm:ssZ`

> **注 :** 簡単にするために、このデータ・エンティティには定義されたリレーションシップはありません。リレーションシップは
> `type=Relationship` で与えられなければなりません。リレーションシップは次のチュートリアルで議論されます。

<a name="defining-properties-of-properties-within-the-ngsi-ld-entity-definition"></a>

### NGSI-LD エンティティ定義内の Properties-of-Properties の定義

_Properties-of-Properties_ (プロパティのプロパティ) は、メタデータ (つまり、_"data about data"_ (データに関するデータ)) と
同等の NGSI-LD です。精度、プロバイダ、使用する単位など、属性値自体のプロパティを記述するために使用されます。
いくつかの組み込みメタデータ属性がすでに存在していて、これらの名前は予約されています :

-   `createdAt` (type: DateTime): ISO 8601 文字列としての属性作成日
-   `modifiedAt` (type: DateTime): ISO 8601 文字列としての属性変更日

さらに `observedAt`, `datasetId`, `instanceId` がオプションで追加される場合もあります。そして `location`,
`observationSpace`, `operationSpace` は Geoproperties にとって特別な意味を持ちます。

上記の例では、メタデータの1つの要素 (つまり、_property-of-a-property_ (プロパティのプロパティ)) が `address` 属性の中に
あります。`verified` フラグは、住所が確認されたかどうかを示します。最も一般的な _property-of-a-property_ は `unitCode`
で、これは 測定単位の UN/CEFACT
[Common Codes](http://wiki.goodrelations-vocabulary.org/Documentation/UN/CEFACT_Common_Codes)
を保持するために使用されるべきです。

<a name="querying-context-data"></a>

## コンテキスト・データのクエリ

消費アプリケーション (consuming application) は、Orion Context Broker に NGSI-LD HTTP リクエストを送信することで、
コンテキスト・データをリクエストできるようになりました。既存の NGSI-LD インタフェースを使用すると、複雑なクエリを
実行して結果をフィルタ処理し、FQNs または短い名前でデータを取得できます。

<a name="obtain-entity-data-by-fqn-type"></a>

### FQN タイプによるエンティティ・データの取得

この例では、コンテキスト・データ内のすべての `Building` エンティティのデータを返します。`type` パラメータは NGSI-LD
では必須であり、レスポンスをフィルタリングするために使用されます。JSON-LD コンテンツを取得するには、Accept HTTP
ヘッダが必要です。

#### 4️⃣ リクエスト :

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -H 'Accept: application/ld+json' \
  -d 'type=https%3A%2F%2Furi.fiware.org%2Fns%2Fdata-models%23Building'
```

#### レスポンス :

レスポンスはデフォルト (`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld`)
でコアの `@context` を返し、すべての属性は可能な限り展開されます。

-   `id`, `type`, `location`, `name` はコア・コンテキストで定義されており、展開されません
-   `address` は `http://schema.org/address` にマッピングされました
-   `category` は `https://smart-data-models.github.io/data-models/terms.jsonld#/definitions/category` にマッピングされました

エンティティの作成時に属性が FQN に関連付けられていない場合は、短い名前が**常**に表示されます。

```json
[
    {
        "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld",
        "id": "urn:ngsi-ld:Building:store001",
        "type": "https://uri.fiware.org/ns/dataModels#Building",
        "https://schema.org/address": {
            "type": "Property",
            "value": {
                "streetAddress": "Bornholmer Straße 65",
                "addressRegion": "Berlin",
                "addressLocality": "Prenzlauer Berg",
                "postalCode": "10439"
            },
            "verified": {
                "type": "Property",
                "value": true
            }
        },
        "https://uri.etsi.org/ngsi-ld/name": {
            "type": "Property",
            "value": "Bösebrücke Einkauf"
        },
        "https://smart-data-models.github.io/data-models/terms.jsonld#/definitions/category": {
            "type": "VocabProperty",
            "vocab": "https://uri.fiware.org/ns/dataModels#commercial"
        },
        "location": {
            "type": "GeoProperty",
            "value": {
                "type": "Point",
                "coordinates": [
                    13.3986,
                    52.5547
                ]
            }
        }
    },
    {
        "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "https://uri.fiware.org/ns/dataModels#Building",
        "https://schema.org/address": {
            "type": "Property",
            "value": {
                "streetAddress": "Friedrichstraße 44",
                "addressRegion": "Berlin",
                "addressLocality": "Kreuzberg",
                "postalCode": "10969"
            },
            "verified": {
                "type": "Property",
                "value": true
            }
        },
        "https://uri.etsi.org/ngsi-ld/name": {
            "type": "Property",
            "value": "Checkpoint Markt"
        },
        "https://smart-data-models.github.io/data-models/terms.jsonld#/definitions/category": {
            "type": "VocabProperty",
            "vocab": "https://uri.fiware.org/ns/dataModels#commercial"
        },
        "location": {
            "type": "GeoProperty",
            "value": {
                "type": "Point",
                "coordinates": [
                    13.3903,
                    52.5075
                ]
            }
        }
    }
]
```

<a name="obtain-entity-data-by-id"></a>

### ID によるエンティティ・データの取得

この例は `urn:ngsi-ld:Building:store001` のデータを返します

#### 5️⃣ リクエスト :

```console
curl -G -X GET \
   -H 'Accept: application/ld+json' \
   'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001'
```

#### レスポンス :

レスポンスはデフォルト
(`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld`)
で コアの `@context` を返し、すべての属性は可能な限り展開されます。

```json
{
    "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "https://uri.fiware.org/ns/dataModels#Building",
    "https://schema.org/address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "https://uri.etsi.org/ngsi-ld/name": {
        "type": "Property",
        "value": "Bösebrücke Einkauf"
    },
    "https://smart-data-models.github.io/data-models/terms.jsonld#/definitions/category": {
        "type": "VocabProperty",
        "vocab": "https://uri.fiware.org/ns/dataModels#commercial"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        }
    }
}
```

<a name="obtain-entity-data-by-type"></a>

### タイプによるエンティティ・データ取得

提供されたデータへの参照が提供されている場合、短い名前のデータを返し、レスポンスを特定のデータの `type`
に制限することが可能です。例えば、以下のリクエストはコンテキスト・データ内のすべての  `Building` エンティティのデータを
返します。`type` パラメータの使用は `Building` エンティティのみにレスポンスを制限します、`options=keyValues`
クエリ・パラメータの使用はレスポンスを標準の JSON-LD まで減らします。

短い形式の `type="Building"` を FQN `https://uri.fiware.org/ns/dataModels#Building` に関連付けるために
[`Link` ヘッダ](https://www.w3.org/wiki/LinkHeader) を供給しなければなりません。
フル・リンク・ヘッダの構文は次のとおりです :

```text
Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json
```

標準の HTTP の `Link` ヘッダは、実際に問題のリソースに触れることなくメタデータ (この場合は `@context`)
を渡すことを可能にします。NGSI-LD の場合、メタデータは `application/ld+json` 形式のファイルです。

#### 6️⃣ リクエスト :

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'options=keyValues'
```

#### レスポンス :

`options=keyValues` を使用しているため、レスポンスは JSON のみで構成され、属性の定義 `type="Property" ` や
_properties-of-properties_ 要素は含まれていません。リクエストからの `Link` ヘッダはレスポンスに返される `@context`
として使われていることがわかります。

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        }
    },
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="filter-context-data-by-comparing-the-values-of-an-attribute"></a>

### 属性値を比較してコンテキスト・データをフィルタ

この例は `name` 属性 _Checkpoint Markt_ を持つ全ての `Building` エンティティを返します。フィルタリングは `q`
パラメータを使って行うことができます。文字列がスペースを含む場合、それは URL エンコードされ、単一引用符 `"` = `%22`
の中に保持されます。

#### 7️⃣  リクエスト :

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=name==%22Checkpoint%20Markt%22' \
    -d 'options=keyValues'
```

#### レスポンス :

`Link` ヘッダ `https://schema.lab.fiware.org/ld/context` は以下に示すように `@context` の配列を保持します :

```json
{
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ]
}
```

したがって、FIWARE Building モデルが含まれています。

これは、`Link` ヘッダと `options=keyValues` パラメータを使用すると、以下に示すように短い形式の JSON-LD
へのレスポンスが減ることを意味します :

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="filter-context-data-by-comparing-the-values-of-an-attribute-in-an-array"></a>

### 配列内の属性値を比較してコンテキスト・データをフィルタ

標準的な `Building` モデルの中では、`category` 属性は文字列の配列を参照します。この例では、`commercial` または `office`
の文字列を含む `category` 属性を持つすべての `Building` エンティティを返します。フィルタリングは `q`
パラメータを使って行うことができ、コンマは許容値を区切ります。

#### 8️⃣  リクエスト :

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=category==%22commercial%22,%22office%22' \
    -d 'options=keyValues'
```

#### レスポンス :

レスポンスは、短い形式の属性名を使用してJSON-LD 形式で返されます :

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        }
    },
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="filter-context-data-by-comparing-the-values-of-a-sub-attribute"></a>

### サブ属性の値を比較してコンテキスト・データをフィルタ

この例では、Kreuzberg (クロイツベルク) 地区にあるすべてのストアを返します。

フィルタリングは `q` パラメータを使って行うことができます。サブ属性はブラケット構文を使って注釈を付けられます。
例えば `q=address[addressLocality]=="Kreuzberg"`。これは、ドット・シンタックスが使用されていた NGSI v2 とは異なります。

#### 9️⃣ リクエスト :

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=address%5BaddressLocality%5D==%22Kreuzberg%22' \
    -d 'options=keyValues'
```

#### レスポンス :

`Link` ヘッダと `options=keyValues` パラメータの使用は JSON-LD へのレスポンスを減らします。

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="filter-context-data-by-querying-metadata"></a>

### メタデータをクエリすることでコンテキスト・データをフィルタ

この例は検証されたアドレスを持つすべての `Building` エンティティのデータを返します。`Verified` 属性は
_Property-of-a-Property_ の例です

メタデータ・クエリ (すなわち Properties of Properties) はドット・シンタックスを使用して注釈付けされます。
例えば `q=address.verified==true`。これは NGSI v2 の `mq` パラメータに代わるものです。

#### 1️⃣0️⃣ リクエスト :

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=address.verified==true' \
    -d 'options=keyValues'
```

#### レスポンス :

Accept HTTP ヘッダと一緒に `options=keyValues` を使用しているため、レスポンスは属性 `type` と `metadata`
要素のない JSON のみで構成されています。

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
        }
    },
    {
        "@context": "https://fiware.github.io/data-models/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="filter-context-data-by-comparing-the-values-of-a-geojson-attribute"></a>

### geo:json 属性の値を比較してコンテキスト・データをフィルタ

この例では、**Berlin** の **Brandenburg Gate** (_52.5162N 13.3777W_) から2km以内のすべてのストアを返します。
ジオ・クエリ・リクエストをするには、`geometry`, `coordinates`, `georel` の3つのパラメータを指定しなければなりません。

NGSI-LD の構文が更新され、`coordinates` パラメータは、現在では、NGSI v2 で必要とされる単純な緯度と経度ペアではなく、
角括弧 (square brackets) を含む [geoJSON](https://tools.ietf.org/html/rfc7946) で表されるようになりました。

geo-query は デフォルトで `location` 属性に適用されることに注意してください。これは、NGSI-LD でデフォルトで
指定されているためです。他の属性を使うのであれば、追加の `geoproperty` パラメータが必要です。

#### 1️⃣1️⃣ リクエスト :

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -H 'Link: <https://fiware.github.io/data-models/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/ld+json' \
  -d 'type=Building' \
  -d 'geometry=Point' \
  -d 'coordinates=%5B13.3777,52.5162%5D' \
  -d 'georel=near%3BmaxDistance==2000' \
  -d 'options=keyValues'
```

#### レスポンス :

Accept HTTP ヘッダと一緒に `options=keyValues` を使用しているため、レスポンスは属性 `type` と `metadata`
要素のない JSON のみで構成されています。

```json
[
    {
        "@context": [
            "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
        ],
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
        }
    }
]
```

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2019-2025 FIWARE Foundation e.V.
