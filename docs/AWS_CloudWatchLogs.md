
## amazon-cloudwatch-agentインストール

```
sudo yum install amazon-cloudwatch-agent
```

## amazon-cloudwatch-agent設定

```
vi /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```

## amazon-cloudwatch-agent設定反映(tomlファイル作成)

```
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```

## amazon-cloudwatch-agentログ参照

```
tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

## AWS上での設定
### ロググループの作成
amazon-cloudwatch-agent.jsonで指定したロググループ名を指定
## ログストリームの登録
amazon-cloudwatch-agent.tomlに記載されたログストリームを指定

## メトリクスフィルターを指定
 - フィルタパターン
   - 絞りたい文字列
 - フィルタ名
   - 任意
 - メトリクス名前空間
   - 新規作成にしないとNextが押せない
 - メトリクス名
   - ユニークであること
 - メトリクス値
   - [1]
 - デフォルト値
   - [0]

## アラーム設定
 - メトリクス選択
   - 作成したメトリクスフィルターを指定
 - 条件
   - アラームを受取りたい条件
 - 通知
   - 任意
 - Auto Scaling アクション
 - EC2 アクション
 - Systems Manager OpsCenter アクション
   - 指定しない
 - アラーム名
 - アラームの説明
   - 任意


## 参考文献
https://blog.serverworks.co.jp/tech/2020/01/28/cloudwtach-agent-ssm-centos/


# ステータス確認
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
{
  "status": "running",
  "starttime": "2018-10-01T16:19:41+0000",
  "version": "1.203420.0"
}

# 停止
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
amazon-cloudwatch-agent stop/waiting

# 起動
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
amazon-cloudwatch-agent start/running, process 3044

CloudWatchAgentServerPolicy
→EC2にアタッチが必要

https://qiita.com/hayao_k/items/d983177510b3b3a69561


https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html


JVM結果をcloudwatchlogsに送るshell
https://blog.serverworks.co.jp/jvm-gc-report
