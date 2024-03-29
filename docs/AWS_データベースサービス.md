# データベースサービス

## Amazon データベースサービス

- RDS:RDB型のDB
- Redshift:ペタバイト級のデータが扱える
- DynamoDB:KV型のNoSQLDB
- ElasiCache:フルマネージド型のインメモリDB

## AWS_RedShift

- ペタバイト級のデータを扱えるDWH
- Leaderノード一つと複数のComputeノードから成るクラスタ構成で、Computeノードを増やすことにより性能向上を図る

- Leaderノード（リーダーノード）
  - 各クラスタに一台だけ存在する司令塔（Leader）となるノード
  - アプリケーションなどからのリクエストを受け付けて各Computeノードへ割り振う
  - その結果を取りまとめてアプリケーションへ返す
- Computeノード（コンピュートノード）
  - Leaderノードからの命令を処理するノード
  - 一つのComputeノードはCPU、メモリ、ストレージを搭載している
  - Computeノードを増やすことでパフォーマンスを向上させる

### 特徴

- 列指向型データベース(カラムナ)
  - 集計処理に使われるデータを列単位でまとめて処理することで効率化を図る
- エンコード方式
  - 9種類のエンコード方式に対応
    - IOボトルネックを発生させないために種類を多く提供
- ゾーンマップ
  - ブロック単位でデータ格納(1MB/ブロック)
  - ブロック内に格納されているデータの最小値、最大値をメモリに保存する仕組み
    - データ検索の条件に該当する値の有無の判断に利用
    - データが存在しない場合はスキップする
- 拡張性
  - MPP(Massively Parallerl Processing)
    - 1回の集計処理を複数のノードに分散して実行する
      - スケールアウトでパフォーマンス向上できる
  - シェアードナッシング
    - 各ノードがディスクの共有をせず、ノードとディスクセットを拡張する仕組み
- ワークロード(WLM)
  - wlm_json_configurationにパラメータのクエリ定義をする
- Redshift Spectrum
  - S3上のデータをRedshiftの外部テーブルとして参照できるようにした機能
  - S3のデータをいったんRedshiftへ取り込んでからアクセスを行うよりも高速にアクセスできる
    - S3からRed Shiftへのデータロード(COPY)は時間がかかる
    - データ追加に伴いRedshiftクラスタの追加だとコストがかかる(CPUやメモリも増やすため)

### 利用例

- 現在数百テラバイトのデータを保持している。過去3年に蓄積したこのデータを分析し、今後の経営計画に活かしたい
- 構築済みのAmazon Redshiftクラスタを別のリージョンで速やかに再構築できるようにしたい
  - 構築済みのRedshiftクラスタに対してクロスリージョンスナップショットを作成する
  - RedshiftのディスクイメージをAmazon S3へ保存
- Amazon Redshiftクラスタを持つ企業がある。この企業はいくつかの災害対策を検討しており、クラスタのバックアップも課題の一つである。ソリューションアーキテクトがとるべき手段はどれか。
  - Redshiftクラスタのクロスリージョンスナップショットを作成する
- ある企業は、構造化したデータセットと、構造化していないデータセットをAmazon S3上に保持している。データサイズはペタバイト規模で、既存のBIツールを使った分析を今後も継続して行いたい。

## Amazon DynamoDB

- データ構造
  - Key-Value型
    - 既存のデータ構造に変更を加えることなく新しい項目を追加することができる
    - シンプルな構造であるためパフォーマンスは非常に高く、ピーク時には秒間2000万件のリクエストに対応
    - ストレージの容量制限もありませんので拡張性にも優れている
  - 「スキーマレス」のデータベースなので、スキーマ(データベースの構造)の更新に柔軟に対応できる
    - スキーマが存在しないので変更作業自体が不要
- キャパシティ
  - オンデマンドモード
    - テーブルに対する読み込み・書き込みのリクエスト単位で料理料金がかかる
  - プロビジョニングモード
    - キャパシティユニットという単位で読み書きが管理される
      - 1秒間にどれだけ読み込み(RCU)・書き込み(WCU)を行うかを予約する設定で、容量が大きいほど料金がかかる
      - テーブル作成時にはWCU、RCUともに 5 に設定されている(デフォルト)
      - キャパシティユニットの変更はいつでも行うことができる
      - 自動的にスケール（増減）することもできる
    - 大規模に利用する場合は「リザーブドキャパシティ」
      - WCU、RCUともに100ユニット単位で1年または3年の期間で予約購入するサービス
      - 通常のプロビジョニングモードよりも割安に利用可能
- DynamoDB Streams
  - テーブルに対して行われた直近の24時間の変更（追加や更新、削除）をログに保存する機能
  - ストリームを参照することによって、いつ誰がどのようにテーブルを更新したかがわかる
- DynamoDB Accelerator（DAX）
  - DynamoDBのインメモリキャッシュクラスタのこと
  - ミリ秒（千分の一秒）だったレスポンスをマイクロ秒（百万分の一秒）レベルのパフォーマンスにまで向上させることができう

### バックアップ方法

- オンデマンドバックアップ
  - ユーザーが任意のタイミングで作成するバックアップ
  - マネジメントコンソールまたはAPIを利用して、テーブルの完全なバックアップを作成する
- ポイントインタイムリカバリ（PITR）
  - 自動的にバックアップを行いたい場合に有効にする
  - 差分バックアップが定期的に自動で取得される
  - リカバリ時は、35日前まで遡ることができます。
- バックアップ処理もパフォーマンスへ影響を与えることはありません。

## ElastiCache

- インメモリ型DB
- フルマネージド型で、データベースのセットアップや監視・ソフトウェアへのパッチ適用などが自動で行われる
- KVS型のデータベースエンジンを持つ
- すべてのデータをメモリ上に保持する
  - クエリ結果をキャッシュすることにより、パフォーマンスを向上させることができる
- NoSQLのデータベースサービス
  - 例:Amazon RDSでRDBを扱い、ElastiCacheはRDSのキャッシュとして利用する、という連携は可能

### データベースエンジン

- Memcached
  - デファクトスタンダード
  - ストレージへのバックアップ機能やフェイルオーバー機能などがなく、キャッシュとしてのシンプルな利用に向いている
  - データの永続保持、レプリケーション機能はない
    - 万が一消えてもシステム動作に影響しない
  - 必要なキャッシュリソースの増減が頻繁
  - 機密性に関する設定はない
  - 最大20インスタンス構成
  - クラスタ内に保存されるデータは各インスタンスに分散
  - クラスタを作成するとアクセスエンドポイントが作成される
    - ノーエンドポイント
      - クラスタ内の各ノードに個別アクセスするためのエンドポイント
    - 設定エンドポイント
      - クラスタ全体に割り当てられるエンドポイント
      - ノードの増減管理
      - クラスタ構成情報を自動更新する
      - アプリケーションからElastiCacheサービスを使うときはこのエンドポイントを利用する
  - スケーリング
    - スケールIO時、ノード増減によるキャッシュミスが一時的に増えることがある
    - スケールダウン字、新規クラスタを用意する必要があるため、元のデータが消える
- Redis
  - サービスの可用性を上げるために行える施策に向いている
  - 自動フェイルオーバーを備えたマルチAZを設定やバックアップリストア、データ暗号化等ができる
  - データの暗号化や、SSL/TLSによる通信の暗号化、クライアントをパスワードで認証するRedis認証を行える
  - 文字列、リスト、セット等多様なデータ型を使いたい
  - キーストアを永続的に持ちたい
  - クラスタモード
    - 無効
      - 1つのインスタンスに保存
      - リードレプリカは最大5つ
    - 有効
      - 最大15シャードにデータ分割して保存する
      - 1シャードに対してリードレプリカは最大5つ
  - エンドポイント
    - 
