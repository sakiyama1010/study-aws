# アプリケーションサービス

- AWSが提供するサーバリソース上で動いている
  - サーバとアプリケーションのメンテナンスはAWS側が行う
  - コスト、可用性の観点をあまり考えなくてよい

## Amazon Simple Queue Service（SQS）

- フルマネージドのメッセージキューイングサービス
  - リクエストを投げたい側（はリクエストを受け付ける側に対するリクエストをSQSへ送信する
  - リクエストを受け付ける側はSQSにあるリクエストを取得して処理する
  - AWS提供のフルマネージドサービスだと最古(2004年)
  - キュー管理とメッセージ管理の2種類
    - キュー管理
      - エンドポイントを介して使うようになる
      - キュー作成、削除、動作属性の設定
    - メッセージ管理
      - キューに対するメッセージ送信、取得
      - 処理済みのメッセージ削除
      - 複数キューをまとめて処理するバッチ用API
      - 処理中に他のプロセスから取得できなくする可視性制御のAPI
- メッセージキューイング
  - 個々のサービスやシステムを「メッセージ」を使用して連携する仕組みのこと
  - お互いのサービスがそれぞれの状況に
  - 関係なく（非同期で）処理を進行できる
    - **サービス間の結合度は低くなる**
  - キューのメッセージは受信側が明示的に削除指示をしないと削除されない

### キュー

- キュー種類
  - 標準(スタンダード)
    - メッセージの処理順序は保証されず、送信された順番と異なることがある
    - タイミングによっては同じメッセージが二度配信される可能性がある
      - 上記が起きた時の処理は、受信側で制御する必要がある（べきとう性の担保）
    - FIFOキューよりもスループットが優れている
  - FIFO（First In First Out）
    - 送信された順番にキューに残っている最も古いものからメッセージを処理する
      - 受信側はメッセージが送信された順番に受信できる
    - 二重取得することもない

### ポーリング

- ショートポーリング
  - デフォルト
  - キューにメッセージがない場合でも**即座**に空の応答を返す
    - 複数キューを単一スレッドで処理するケース
- ロングポーリング
  - メッセージがない場合は、設定されたタイムアウトのギリギリまでレスポンスを返さない
    - メッセージを受信するまでの時間を設定できる
    - キューが空の応答が減りAPIコール数を抑制できる

### 可視性タイムアウト(Visibility TimeOut)

- 同じメッセージの受信を防止する機能
  - 受信中にほかの人がメッセージを受信するのを防止
  - デフォルトは30s(MAX12h)

### 遅延キュー・メッセージタイマー

- メッセージの配信時間をコントロールする機能
- 遅延キュー
  - **キュー全体に対して**に送られたメッセージを一定時間見えなくする
- メッセージタイマー
  - **個別のメッセージに対して**一定時間見えなくする
  - メッセージのデフォルト（最小）遅延は0秒、最大数は15分
  - メッセージタイマー設定は、 Amazon SQS遅延キューのすべてのDelaySeconds 値よりも優先される

### デッドレターキュー

- 処理できないメッセージを別のキューに移動する機能
  - エラーが一定の回数に達したメッセージを隔離するためのキュー
  - エラーになるようなメッセージがあった場合、メッセージはキューに残り続ける
    - その場合キューにはメッセージが滞留し、受信側ではエラーになる処理をリトライし続けてしまう
- 使用例
  - Webシステムでは、有料会員の処理は優先して行い、無料会員は低優先で行うようにしたい。Amazon SQSを使って実装するとき、ソリューションアーキテクトが推奨するキューの構成
    - 2つのキューを作成し、優先度ごとにキューを使い分ける。メッセージの受信側は優先度の高いキューを先に処理するようにする

### メッセージサイズ

- メッセージの最大サイズは256KB
  - 大きいデータ(画像とか)はS3やDynamoDB等に入れてパスやキーを渡して対処する

## ワークフローサービス

- マネージド型のタスクコーディネータ
- 商品の発注/請求処理のワークフロー（処理の流れ）のような、重複が許されない、厳密に一回限り順序性が求められる処理のコーディネータとしての利用に適している
- ワークフロースターター
  - SWFを起動し、ワークフロー開始の起点となるアプリケーション

### AWS Step Functions

- SWFと同様機能だが、ワークフローを可視化し編集できる機能がある

## Amazon Simple Notification Service（SNS）

- プッシュ型のメッセージングサービス
  - ユーザーやアプリケーションに対してメッセージ送信を行うサービス
- システムの状態変化があった際,のシステム管理者への通知や不特定多数のユーザーへの定期的なメッセージ配信などを、EメールやSMS、Amazon SQSを通してユーザーやアプリケーションに対して通知する
- 通知方法（プロトコル）
  - Eメール
  - HTTP/HTTPS
  - モバイル（iOS、Android）端末
  - SMS
  - Amazon SQSへキューを登録
  - AWS Lambdaと連携してLambda関数を呼び出す
- トピック
  - 情報の管理単位
    - 管理者はトピックを作成する
  - 購読者（Subscriber）
    - 購読者がトピックを使ってメッセージ受信を行う
      - 通知を受け取りたいトピックを選択する
      - 受け取りたいメッセージの形式（プロトコル）を選ぶ
  - 発行者（Publisher）
    - 発行者がトピックを使ってメッセージ配信を行う
    - メッセージを発行したいトピックを選択してメッセージを配信する
    - メッセージの発行者は、プロトコルやメッセージの通知先（購読者：Subscriber）の存在などを意識せずにメッセージを発行できる
      - 複数のアプリケーション間を低い結合度（疎結合）で連携できる
- 利用例
  - EC2インスタンスの状態が変化した際、自動的にLambda関数が呼び出されるようにしたい
    - EC2インスタンスの状態を通知するためのSNSトピックを作成し、購読者（Subscriber）としてLambda関数を登録する
  - AWS Configルールに違反している使われ方をした
  - CloudWatchでEC2のCPU、ネットワークなどのメトリクスを監視しているときに閾値を超えた

## Amazon Simple Email Service（SES）

- フルマネージド型のEメール送受信サービス
  - 受信したメールをS3に保存することもできる
  - 送信時にウイルスやマルウェアを検出しブロックする機能
  - 送信成功数や拒否数を統計的にsy理いｓ、配信不能な苦情を管理する機能
  - SPF,DKIMでの認証機能
- バウンス率や苦情率を注視し、常に低く保つ運用を行う必要がある
  - Bounceメールの比率を5%以下を保ち続ける
  - 苦情を防ぐ(0.1%未満)
  - 悪意のあるコンテンツを送らない
- メール受信をトリガーにAWS LambdaのLambda関数を実行することも可能

## その他

- 疎結合
  - システムを構成するサービスやコンポーネント間の結合度や依存度を低くし、独立性を高めること
    - 「デカップリング」(切り離し)とも言う
  - 結合度を低くする
    - 関連するサービスの負荷や処理状況に引きずられずに自身の処理を進行できる
    - 関連するサービスで障害が発生した場合やアップデートなどの際に影響を受けずに済む
- ポーリング（polling）
  - 機器などに対して、一定間隔で順番に問合せ（データの送信要求など）を行うこと
- スループット
  - 一般に単位時間当たりの処理能力やデータ転送量のこと
