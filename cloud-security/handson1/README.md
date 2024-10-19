# 代表的なセキュリティサービスの操作

## CloudTrailの有効化

証跡を作成する
- ログファイルのSSE−KMS暗号化: 無効
- ログファイルの検証: 有効

## GuardDutyの有効化

GuardDutyの有効化をするだけ
検知設定はデフォルト

## AWS Configの有効化

Configを有効化する
- 記録戦略: カスタマイズ可能なオーバーライドのあるすべてのリソースタイプ
- 記録頻度: 継続的な記録 <- コストと相談かな
- オーバーライド設定: [グローバルに記録されたすべてのIAMリソースタイプ: 記録から削除]

※グローバルに記録されたすべてのIAMリソースタイプはどこか1箇所のリージョンを除いて記録から削除しておいて方が良さそう

## Security Hubの有効化

ecurity Hubの有効化

- セキュリティ基準
- AWS基礎セキュリティのベストプラクティスv1.0.0 : 有効
- CIS AWS Foundations Benchmark v1.4.0 : 有効

## IAM Access Analyzerの有効化

アナライザーを作成
- 検出結果タイプ: 外部アクセス分析


# 検出結果の確認

- GuardDutyの検出結果のサンプルを出力
    - 設定から`検出結果サンプルの生成`を選択

[Amazon GuardDuty の検出結果について](https://docs.aws.amazon.com/ja_jp/guardduty/latest/ug/guardduty_findings.html)

- CloudTrailの証跡の確認にCloudTrail Lakeを使用することでSQLベースのクエリで検索できるようになる

[ClouTrail Lake の使用](https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/cloudtrail-lake.html)

