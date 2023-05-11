<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　Redshift で3世代分を残すようなスナップショットの取得方法がやりたかったので、その備忘録です。  

　本記事のコマンドはBash上で実行しています。

## 実現方法

### スナップショットを取得

　`aws redshift create-cluster-snapshot`コマンドを使用することで、Redshiftのスナップショットを取得することが可能です。

<div class="md-code" style="width:100%">
```sh
aws redshift create-cluster-snapshot --region "ap-northeast-1" \
--cluster-identifier "sample-cluster" --snapshot-identifier "sample-cluster-2018-12-20-00-00-00"
```
</div>

### スナップショットの削除

　`aws redshift describe-cluster-snapshots` コマンドを使用することでスナップショットの一覧を取得することができます。  

下記の例では、取得した一覧で最古のスナップショットを `aws redshift delete-cluster-snapshot` コマンドで削除しています。

<div class="md-code" style="width:100%">

```sh
# 指定したクラスターの最古のスナップショットIDを取得する
OLDEST_SNAPSHOT_ID=$(aws redshift describe-cluster-snapshots --region "ap-northeast-1" --cluster-identifier "sample-cluster" --snapshot-type "manual" \
--query "sort_by(Snapshots[*].{SnapshotIdentifier:SnapshotIdentifier, SnapshotCreateTime:SnapshotCreateTime}, &SnapshotCreateTime)[0].SnapshotIdentifier" --output text)

# 取得したスナップショットIDを指定して、スナップショットを削除
aws redshift delete-cluster-snapshot --region "ap-northeast-1" --snapshot-identifier "${OLDEST_SNAPSHOT_ID}"
```

</div>

### 取得と削除をまとめる

　最終的には下記のようなシェルになりました。スナップショットを取得し、その後に削除するような処理です。  

クラスターに紐づくスナップショットが既に3つある場合には、3世代分を残しておくような取得方法になります。

<div class="md-code" style="width:100%">

```sh
#!/bin/bash

TIMESTAMP=`date '+%Y-%m-%d-%H-%M-%S'`
REGION="ap-northeast-1"
CLUSTER_NAME="sample-cluster"

# スナップショットを取得
aws redshift create-cluster-snapshot --region "${REGION}" --cluster-identifier "${CLUSTER_NAME}" --snapshot-identifier "${CLUSTER_NAME}-${TIMESTAMP}"

# 最古のスナップショットを削除
OLDEST_SNAPSHOT_ID=$(aws redshift describe-cluster-snapshots --region "ap-northeast-1" --cluster-identifier "${CLUSTER_NAME}" --snapshot-type "manual" \
--query "sort_by(Snapshots[*].{SnapshotIdentifier:SnapshotIdentifier, SnapshotCreateTime:SnapshotCreateTime}, &SnapshotCreateTime)[0].SnapshotIdentifier" --output text)
aws redshift delete-cluster-snapshot --region "${REGION}" --snapshot-identifier "${OLDEST_SNAPSHOT_ID}"
```

</div>

## 参考

- <span><a href="https://docs.aws.amazon.com/cli/latest/reference/redshift/index.html" target="_blank">redshift — AWS CLI 1.19.30 Command Reference</a></span>
