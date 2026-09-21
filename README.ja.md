<h1  align="center">
    <img src="https://fiware.github.io/tutorials.Step-by-Step/img/fiware-farm.png" />
    <img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"/>
    <br/>
    👨‍🌾 👩‍🌾 🐄 🐐 🐑 🐖 🐓 🌻 🥕 🌽
</h1>

## Linked Data

[![FIWARE Core Context Management](https://fiware.github.io/catalogue/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

このチュートリアルでは、既存の **NGSI-v2** Context Broker を **NGSI-LD** データ・スペース内の Context Source として
使用するという考え方を紹介し、固定された `@context` を持つシンプルな **NGSI-LD** ファサード・パターンを導入することで、
JSON ベースと JSON-LD ベースの両方のコンテキスト・データを統一された構造にまとめる方法を示します。このチュートリアルは、
元の **NGSI-v2** 入門チュートリアルのデータを再利用しますが、**NGSI-LD** インタフェースの API 呼び出しを使用します。

チュートリアルでは全体で [cUrl](https://ec.haxx.se/) コマンドを使用しますが、
[Postman documentation](https://fiware.github.io/tutorials.Linked-Data/ngsi-ld.html)
ドキュメントとしても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/125db8d3a1ea3dab8e3f)
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?repo=FIWARE/tutorials.Linked-Data&ref=NGSI-LD)

> [!NOTE]
>
> このチュートリアルは、既存の **NGSI-v2** システムを **NGSI-LD** フェデレーションに接続しようとしている既存の
> **NGSI-v2** 開発者向けに設計されています。リンクト・データ・システムをゼロから構築する場合は、
> [NGSI-LD developers tutorial](https://ngsi-ld-tutorials.readthedocs.io/) ドキュメントの最初から始めてください。
>
> 同様に、既存の **NGSI-v2** 開発者で、リンクト・データの概念全般を理解したいだけの場合は、**NGSI-v2** データ・ソースを
> **NGSI-LD** にアップグレードする方法についての同等の
> [**NGSI-v2** チュートリアル](https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-v2) を参照してください

## コンテンツ

# FIWARE データ・エンティティへのリンクト・データ概念の追加

> "Always be a first rate version of yourself and not a second rate version of someone else."
>
> ― Judy Garland

FIWARE の最初の入門である [Getting Started tutorial](https://github.com/FIWARE/tutorials.Getting-Started) では、
コンテキスト・データ・エンティティの作成と操作に一般的に使用される
[NGSI v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) の JSON ベースのインタフェースを紹介しました。
[NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
は、**リンクト・データ** (linked data) の概念を追加することでコンテキスト・データ・エンティティを強化した、
そのインタフェースの進化形です。

2つの別々のインタフェースが必要な理由は次のとおりです:

-   **NGSI-v2** は、個々のスマート・システムで使用される [JSON](https://www.json.org/json-en.html) ベースの相互運用性を
    提供します
-   **NGSI-LD** は、フェデレーションとデータ・スペースで使用される [JSON-LD](https://json-ld.org/) ベースの
    相互運用性を提供します

**NGSI-v2** は、Web サービスや IoT デバイス向けに相互運用可能なインタフェースを提供する個々のアプリケーションを
作成するのに理想的です。NGSI-LD よりも理解しやすく、JSON-LD の `@context` を必要としません。しかし、**NGSI-v2**
はリンクト・データがない場合、データ・スペース全体にフェデレーションされたデータを提供しようとすると不十分です。
複数のアプリケーションや組織が関与するようになると、各データ所有者は、データ・スペース内の他の参加者が内部で使用する
データの構造をもはや制御できなくなります。ここで **NGSI-LD** の `@context` の出番です。これは属性のマッピング・
メカニズムとして機能し、各ローカル・システムが保持するデータ内で独自の用語を引き続き使用できるようにする一方で、
データ・スペース内の他のユーザからのフェデレーションされたデータは、標準の展開/圧縮操作を使用してリネームされ、
各個々のシステムがデータ・スペース全体からのデータを総合的に理解できるようになります。

## 共通データ・スペースの作成

たとえば、2つの参加者がデータをプールする、シンプルな "Buildings" データ・スペースを想像してください:

-   **NGSI-v2** チュートリアルからの一連の[スーパーマーケット](https://fiware-tutorials.readthedocs.io/en/latest/)の
    詳細
-   **NGSI-LD** チュートリアルからの一連の[農場](https://ngsi-ld-tutorials.readthedocs.io/en/latest/)の詳細

2つの参加者は、共通の
[データ・モデル](https://ngsi-ld-tutorials.readthedocs.io/en/latest/datamodels.html#building)を
互いの間で使用することに合意していますが、内部的にはわずかに異なる構造でデータを保持しています。

#### 農場 (**NGSI-LD** データ)

**NGSI-LD** の Smart Farm では、すべての Building エンティティは短縮名として `"type":"Building"` でマークされ、
JSON-LD の展開規則を使用して完全な URI `https://uri.fiware.org/ns/dataModels#Building` に展開できます。各 Building
エンティティのすべての属性 (`name`, `category` など) は、[User `@context`](./data-models/ngsi-context.jsonld) を
使用して定義され、NGSI-LD 属性として構造化されます。標準の NGSI-LD キーワードは、各属性の性質を定義するために
使用されます (例: `Property`, `GeoProperty`, `VocabProperty`, `Relationship`)。これらの用語はそれぞれ、
[core `@context`](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld) 内でも定義されています。

```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "category": {
        "type": "VocabProperty",
        "vocab": "farm"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Großer Stern 1",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3505, 52.5144]
        }
    },
    "name": {
        "type": "Property",
        "value": "Victory Farm"
    },
    "owner": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Person:person001"
    }
}
```

#### スーパーマーケット (**NGSI-v2** データ)

**NGSI-v2** の Smart Supermarket では、すべてのエンティティが `id` と `type` を持たなければなりません - これは
データ・モデルの一部であり、データ・スペースに参加するための前提条件です。しかし、使用されているバックエンド・
システムにより、内部の短縮名は `"type":"Store"` です。**NGSI-v2** システムには `@context` の概念がありません -
すべての属性には `type` と `value` があり、属性の `type` は通常データ型 (`Text`, `PostalAddress` など) に
合わせられますが、`Relationship`, `VocabProperty` のような **NGSI-LD** キーワード型も規約によって許容されて
設定されます。

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "type": "PostalAddress",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        }
    },
    "category": {
        "type": "VocabProperty",
        "value": "supermarket"
    },
    "location": {
        "type": "geo:json",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    "name": {
        "type": "Text",
        "value": "Bösebrücke Einkauf"
    },
    "owner": {
        "type": "Relationship",
        "value": "urn:ngsi-ld:Person:person001"
    }
}
```

### データ・スペースの種類

データ・スペースに参加する際、参加者はそのデータ・スペースを統治するルールに従わなければなりません。共通データ・
スペースが最初に決定しなければならないことの1つは、データ・スペース自体の性質を定義することです。これには主に
3つのアプローチがあり、大まかに次のように定義できます:

-   **統合された** (Integrated) データ・スペースは、すべての参加者が全く同じペイロードとインフラストラクチャを
    使用することを要求します - たとえば _"Scorpio 4.1.15 のみを使用する"_ 。これは、すべての端末が同じ形式で
    チケット・データを送信する必要がある宝くじ発券システムの要件になる可能性があります。
-   **統一された** (unified) データ・スペースは、共通のデータ形式を定義しますが、参加者間で基盤となる
    インフラストラクチャが異なることを許容します - たとえば _"すべてのペイロードに NGSI-LD のみを使用する"_
-   **フェデレーションされた** (federated) データ・スペースはさらに緩やかで、共通のメタ構造を定義するのみで、
    各参加者は基盤となる技術についてより多くの柔軟性を持ちます。たとえば、GIS と NGSI-LD を組み合わせた
    データ・スペースでは "すべてのペイロードは GeoJSON として変換可能でなければならない" \_ などです。

この用語を使うと、この例では参加者がデータ交換に共通して NGSI-LD を使用している一方で、基盤となるシステムは
異なるため、**統一された** データ・スペースを作成していることになります。

# 前提条件

## Docker

物事を単純にするために、すべてのコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。
**Docker** は、さまざまなコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Docker を Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に
    従ってください
-   Docker を Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従って
    ください
-   Docker を Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Linked-Data/master/docker-compose/orion-ld.yml)
は、アプリケーションに必要なサービスを構成するために使用されます。つまり、すべてのコンテナ・サービスを1つの
コマンドで起動できます。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部として
インストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている手順に
従う必要があります。

# アーキテクチャ

デモ・アプリケーションは、データ・スペース内で NGSI-LD 呼び出しを送受信します。このデモ・アプリケーションは、
3つの FIWARE コンポーネントのみを使用します。

現在、Orion と Orion-LD の両方の Context Broker は、コンテキスト・データを永続化するために、オープン・
ソースの [MongoDB](https://www.mongodb.com/) テクノロジに依存しています。したがって、アーキテクチャは
次の2つの要素で構成されます:

-   [Smart Farm Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを受信します
-   [Smart Supermarket Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、
    [NGSI-v2](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを受信します
-   基盤となる [MongoDB](https://www.mongodb.com/) データベースの2つのインスタンス
    -   NGSI-v2 と NGSI-LD の両方の Context Broker が、データ・エンティティ、サブスクリプション、
        レジストレーションなどのコンテキスト・データの情報を保持するために使用します
-   NGSI-LD を NGSI-v2 に、またその逆に変換する FIWARE [Lepus](https://github.com/jason-fox/lepus) プロキシ
-   システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供する HTTP **Web-Server**

2つの要素間のすべての相互作用は HTTP リクエストによって開始されるため、要素をコンテナ化して、公開された
ポートから実行できます。

![](https://fiware.github.io/tutorials.Linked-Data/img/architecture-lepus.png)

必要な構成情報は、関連する `common.yml` および `orion-ld.yml` ファイルの services セクションで確認できます:

#### Orion-LD (NGSI-LD)

```yaml
orion-ld:
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

#### Orion (NGSI-v2)

```yaml
orion:
    image: quay.io/fiware/orion
    hostname: orion2
    container_name: fiware-orion-v2
    depends_on:
        - mongo-for-orion-v2
    networks:
        - default
    ports:
        - "1027:1026" # localhost:1026
    command: -dbURI mongodb://mongo-db2 -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion2:1026/version || exit 1
```

#### Lepus

```yaml
lepus:
    image: quay.io/fiware/lepus
    hostname: adapter
    container_name: fiware-lepus
    networks:
        - default
    expose:
        - "3005"
    ports:
        - "3005:3000"
    environment:
        - DEBUG=adapter:*
        - NGSI_V2_CONTEXT_BROKER=http://orion2:1026
        - USER_CONTEXT_URL=http://context/fixed-context.jsonld
        - CORE_CONTEXT_URL=https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld
        - NOTIFICATION_RELAY_URL=http://adapter:3000/notify
```

すべてのコンテナは同じネットワーク上に存在します - Orion-LD Context Broker はポート `1026` で待ち受け、
最初の MongoDB はデフォルト・ポート `27017` で待ち受けます。Orion NGSI-v2 Context Broker はポート `1027`
で待ち受け、2番目の MongoDB はポート `27018` で待ち受けます。Lepus はポート `3000` を使用しますが、
チュートリアル・アプリケーションと競合するため、外部的にはポート `3005` に変更されています。

Lepus は、以下に示す環境変数によって駆動されます:

| Key                    | Value                                                              | Description                                                                        |
| ---------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| DEBUG                  | `adapter:*`                                                        | ロギングに使用されるデバッグ・フラグ                                              |
| NGSI_V2_CONTEXT_BROKER | `http://orion2:1026`                                               | 基盤となる NGSI-v2 context broker のホスト名とポート                              |
| USER_CONTEXT_URL       | `http://context/fixed-context.jsonld`                              | データ・モデルの定義に使用されるユーザ定義の `@context` ファイルの場所            |
| CORE_CONTEXT_URL       | `https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld` | NGSI-LD エンティティの構造に使用される core `@context` ファイルの場所             |
| NOTIFICATION_RELAY_URL | `http://adapter:3000/notify`                                       | NGSI-v2 から NGSI-LD への通知ペイロードの変換に使用されるコールバック URL         |

# 起動

すべてのサービスは、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Linked-Data/blob/NGSI-LD/services) Bash script を実行することにより、
コマンドラインから初期化できます。以下のコマンドを実行して、リポジトリのクローンを作成し、必要なイメージを
作成してください:

```console
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
git checkout NGSI-LD

./services orion|scorpio|stellio
```

> [!NOTE]
>
> クリーンアップしてやり直す場合は、次のコマンドを実行してください:
>
> ```
> ./services stop
> ```

---

# NGSI-LD データ・スペースの一部として NGSI-v2 エンティティを提供

このチュートリアルは、NGSI-LD Smart Farm (ポート `1027`) と NGSI-v2 Smart Supermarket (ポート `1027`) の
両方を起動し、フェデレーションされたデータ・スペースの一部として、NGSI-v2 エンティティを building finder を
介して Smart Farm FMIS に提供します。

## NGSI-v2 データを直接読み取る

いつものように、公開されたポートに HTTP リクエストを行うことで、スーパーマーケット・データを保持する NGSI-v2
context broker が実行されているかどうかを確認できます。リクエストは NGSI-v2 形式です。`@context` 用の `Link`
ヘッダは必要ありません。

#### 1️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1027/v2/entities/urn:ngsi-ld:Store:001'
```

#### レスポンス:

レスポンスは、以下のように NGSI-v2 形式になります:

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "type": "PostalAddress",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "metadata": {}
    },
    "category": {
        "type": "VocabProperty",
        "value": "supermarket",
        "metadata": {}
    },
    "location": {
        "type": "geo:json",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        },
        "metadata": {}
    },
    "name": {
        "type": "Text",
        "value": "Bösebrücke Einkauf",
        "metadata": {}
    },
    "owner": {
        "type": "Relationship",
        "value": "urn:ngsi-ld:Person:001",
        "metadata": {
            "objectType": {
                "type": "VocabProperty",
                "value": "Person"
            }
        }
    }
}
```

NGSI-v2 の `type` 属性はゆるく定義されていますが、この場合、通常のプロパティを除いて、`Relationship` や
`VocabProperty` のような NGSI-LD で使用される用語に対応するように `type` を使用しています。通常の NGSI-v2
プロパティの場合、`type` は `Text` や `PostalAddress` のようなデータ型に対応し、これらの各データ型は、
NGSI-LD システムでデータを理解できるようにするために JSON-LD `@context` にマッピングする必要があります。

## NGSI-LD 形式で NGSI-v2 データを読み取る

データを NGSI-LD 形式で読み取るには、ポート `3005` で待ち受けている Lepus アダプタを介して NGSI-LD リクエストを
行います。

#### 2️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:3005/ngsi-ld/v1/entities/urn:ngsi-ld:Store:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/ld+json'
```

#### レスポンス:

レスポンスは、以下のように NGSI-LD 形式になります:

```json
{
    "@context": [
        "http://context/fixed-context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
    ],
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        }
    },
    "category": {
        "type": "VocabProperty",
        "vocab": "supermarket"
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
    "owner": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Person:001",
        "objectType": "Person"
    }
}
```

`Link` ヘッダで提供された `@context` は完全に無視され、代わりにアダプタ用に構成された固定の `@context` が
返されていることに注意してください。アダプタは NGSI-LD を完全には理解しておらず、基盤となる NGSI-v2 を
NGSI-LD としてフォーマットしているだけです。NGSI-v2 の `"type": "VocabProperty", "value": "supermarket"` が、
有効な **VocabProperty** である `"type": "VocabProperty", "vocab": "supermarket"` に修正されていることが
わかります。同様に、**Relationship** も、core `@context` で定義されているとおり、`object` と `objectType`
を使用するようになっています。

#### 3️⃣ リクエスト:

`fixed-context` の JSON-LD `@context` ファイルは、NGSI-v2 システム内で見つかったすべての用語を IRI として
完全に定義しています - このファイルは以下のようにアクセスできます。

```console
curl -X GET \
  'http://localhost:3004/fixed-context.jsonld'
```

#### レスポンス:

レスポンスは有効な JSON-LD `@context` です。ご覧のとおり、NGSI-v2 の `Store` は `fiware>Building` に
マッピングされており、これは `https://uri.fiware.org/ns/dataModels#Building` に対応しています。

```json
{
    "@context": {
        "type": "@type",
        "id": "@id",
        "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
        "fiware": "https://uri.fiware.org/ns/dataModels#",
        "schema": "https://schema.org/",
        "tutorial": "https://fiware.github.io/tutorials.Step-by-Step/schema/",
        "openstreetmap": "https://wiki.openstreetmap.org/wiki/Tag:building%3D",
        "Bell": "tutorial:Bell",
        "Store": "fiware:Building",
        "Door": "tutorial:Door",
        "Lamp": "tutorial:Lamp",
        "Motion": "tutorial:Motion",
        "Person": "fiware:Person",
        "Product": "tutorial:Product",
        "Shelf": "tutorial:Shelf",
        "StockOrder": "tutorial:StockOrder",
        "additionalName": "schema:additionalName",
        "address": "schema:address",
        "addressCountry": "schema:addressCountry",
        "addressLocality": "schema:addressLocality",
        "addressRegion": "schema:addressRegion",
... etc
    }
}
```

Lepus アダプタは、NGSI-LD context broker のすべてのエンドポイントを提供することを目指していますが、
JSON-LD `@context` の理解はありません - したがって、すべての用語は常に圧縮されている必要があります -
たとえば、すべての `Stores` に対するクエリは `/ngsi-ld/v1/entities/?type=Store` になります。1.8.1 NGSI-LD
仕様で定義されているエンドポイントの1つに `/info/sourceIdentity` があり、これは基盤となる Context Broker
に関する情報 (稼働時間、エイリアシング、詳細情報) を提供します。この場合、アダプタは Orion broker の
`/version` エンドポイントからのデータを標準化しているだけです:

#### 4️⃣ リクエスト:

```console
curl -G -X GET \
    'http://localhost:3005/ngsi-ld/v1/info/sourceIdentity' \
-H 'Accept: application/ld+json'
```

#### レスポンス:

レスポンスには、アダプタの `contextSourceAlias` (ニックネーム) に加え、ISO 8601 期間形式での稼働時間に
対応するデータおよびサーバ時刻が保持されます。属性 `contextSourceExtras` は、Context Broker に関する
追加の未指定データを保持できるフリーフォームの JSON オブジェクト (`"@type": "@JSON"` として定義) です:

```json
{
    "id": "urn:ngsi-ld:ContextSourceIdentity:16d6f0c6-ce4d-4568-bd0b-c261135354ea",
    "type": "ContextSourceIdentity",
    "contextSourceExtras": {
        "version": "4.0.0",
        "uptime": "0 d, 0 h, 45 m, 58 s",
        "git_hash": "4f9f34df07395c54387a53074f98bef00b1130a3",
        "compile_time": "Thu Jun 6 07:35:47 UTC 2024",
        "compiled_by": "root",
        "compiled_in": "buildkitsandbox",
        "release_date": "Thu Jun 6 07:35:47 UTC 2024",
        "machine": "x86_64",
        "doc": "https://fiware-orion.rtfd.io/en/4.0.0/"
    },
    "contextSourceUptime": "P0DT0H45M58S",
    "contextSourceTimeAt": "2024-08-02T11:16:06.376Z",
    "contextSourceAlias": "lepus",
    "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld"
}
```

### フェデレーション・レジストレーションの作成

この NGSI-LD **ContextSourceRegistration** の例は、アダプタからもデータが利用可能であることを NGSI-LD
context broker に通知します:

#### 5️⃣ リクエスト:

```console
curl -X POST \
 'http://localhost:1026/ngsi-ld/v1/csourceRegistrations/' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d '{
    "type": "ContextSourceRegistration",
    "information": [
        {"entities": [{"type": "Building"}]}
    ],
    "contextSourceInfo": [
        {"key": "jsonldContext","value": "http://context/fixed-context.jsonld"},
        {"key": "Prefer", "value": "ngsi-ld=1.6"}
    ],
    "mode": "inclusive",
    "operations": ["federationOps"],
    "endpoint": "http://adapter:3000"
}'
```

レジストレーションは次のように構成されています:

-   `information.entities` は、`Building` エンティティが `endpoint` で利用可能である可能性があることを
    示しています。`Link` ヘッダは、短縮名 `Building` を IRI `https://uri.fiware.org/ns/dataModels#Building`
    にマッピングする JSON-LD `@context` を保持しています。
-   レジストレーションの `mode` は `inclusive` です。これは、Context Broker が、登録されたすべての
    `Building` データを合算し、ローカルで見つかった `Building` エンティティと組み合わせることを意味します。
-   `operation` モードは、`endpoint` がサブスクリプションとエンティティの取得のみを処理できることを示します。
-   `contextSourceInfo` は、`endpoint` へのリクエストを転送する際に使用されるキー・バリュー・ペアを保持します:
    -   `jsonldContext` は特別なキーで、レジストラントのエンドポイントに転送する前にペイロードに JSON-LD
        圧縮操作を適用するために使用されます。これは Lepus 自身が使用する固定の用語に対応しており、背後の
        NGSI-v2 broker が常に好みの短縮名の用語を提供されることを保証します。
    -   `Prefer` は、転送されるリクエストに `Prefer` ヘッダを追加します。これにより、Lepus は 1.6.1 NGSI-LD
        仕様に対応する要素のみを返すことが保証されます
-   `endpoint` は、NGSI-v2 context broker の前段にあるアダプタの場所を保持します

レジストレーションが設定されると、NGSI-LD FMIS システムに問い合わせることで、NGSI-v2 の Stores に関する
情報を読み取ることができます:

#### 6️⃣ リクエスト:

```console
curl -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Store:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

レジストレーションで `Prefer: ngsi-ld=1.6` が設定されていたため、取得されたエンティティは **VocabProperty**
を持たず、`objectType` はレスポンスから静かに取り除かれています。これにより、NGSI-LD 仕様の以前のバージョン
(この場合はバージョン 1.6) に準拠した Context Broker との後方互換性が保証されます。

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Building",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        }
    },
    "category": { "type": "VocabProperty", "vocab": "supermarket" },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    "name": { "type": "Property", "value": "Bösebrücke Einkauf" },
    "owner": { "type": "Relationship", "object": "urn:ngsi-ld:Person:001" }
}
```

### エンティティの取得

この例では、"Supermarket" `category` を持つ既知のすべての `Building` エンティティを返します

#### 7️⃣ リクエスト:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=category==%22supermarket%22' \
    -d 'pick=id,type,name' \
    -d 'format=simplified'
```

#### レスポンス:

レスポンスは、ローカルで見つかったすべての **Building** エンティティと、登録された Context Broker で
見つかったすべての **Building** エンティティを返します。

```json
[
    {
        "id": "urn:ngsi-ld:Store:001",
        "type": "Store",
        "name": "Bösebrücke Einkauf"
    },
    {
        "id": "urn:ngsi-ld:Store:002",
        "type": "Store",
        "name": "Checkpoint Markt"
    },
    {
        "id": "urn:ngsi-ld:Store:003",
        "type": "Store",
        "name": "East Side Galleria"
    },
    {
        "id": "urn:ngsi-ld:Store:004",
        "type": "Store",
        "name": "Tower Trödelmarkt"
    }
]
```

これは、農場の建物とスーパーマーケットの店舗の両方にアクセスできる `localhost:3000` の FMIS システム自体で
確認できます:

![](https://fiware.github.io//tutorials.Linked-Data/img/buildings.png)

### 代替の `@context` を使用する

Lepus も NGSI-v2 context broker も代替の `@context` ファイルを処理できませんが、NGSI-LD context broker から
データをリクエストする場合、JSON-LD は完全にサポートされているため、ユーザ・エージェントの好みの短縮名を
使用してレスポンスを返すことができます。

以下の例では、**Building** エンティティ `urn:ngsi-ld:Store:001` がドイツ語の用語を使用してリクエストされます。

#### 8️⃣ リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Store:001' \
-H 'Link: <http://context/alternate-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

レスポンスは、ドイツ語の短縮形式の属性名を持つ JSON 形式で返されます。エンティティの真のソース
(アダプタ経由の NGSI-v2 データ) はエンド・ユーザには見えません。

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Gebäude",
    "adresse": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        }
    },
    "kategorie": { "type": "Property", "value": "supermarket" },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    "name": { "type": "Property", "value": "Bösebrücke Einkauf" },
    "inhaber": { "type": "Relationship", "object": "urn:ngsi-ld:Person:001" }
}
```

## License

[MIT](LICENSE) © 2024-2026 FIWARE Foundation e.V.
