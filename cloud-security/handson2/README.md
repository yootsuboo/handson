# セキュリティ対応環境構築、脅威検知/対応

## 前提

- 対象リージョン: us-west-2(オレゴン)
- AWSサービスの有効化
    - GuardDuty
    - Security Hub
        - セキュリティ基準は無効化
    - Config
- Default VPCが存在している

## 環境構築1

- CloudFormationテンプレートの実行
    - s3テンプレート: `https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshop-us-west-2/threat-detect-workshop/staging/01-environment-setup-nom.yml`
    - スタック名: ThreatDetectionWksp-Env-Setup
    - Eメールアドレス: <my-email-address>

- 登録したメールアドレスの許可
    - AWSからのメール内リンク`Confirm subscription`をクリック

- Config Rules 評価結果のメール通知の設定(SNSと連携してメールが送信されるようになる)
    - EventBridgeルールの作成
        - 名前: post_snstopic_configrule
        - イベントパターン
        ```
        {
          "source": ["aws.config"],
          "detail-type": ["Config Rules Compliance Change"]
        }
        ```
        - ターゲット
            - SNSトピック: threat-detection-wksp

- Config Rulesの設定
    - ルールの有効化: s3-bucket-level-public-access-prohibited

## 環境構築2

- CloudFormationテンプレートの実行
    - s3テンプレート: `https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshop-us-west-2/threat-detect-workshop/staging/02-attack-simulation-nom.yml`
    - スタック名: ThreatDetectionWksp-Attacks

- スタック作成完了後40分ほどしたら擬似的な攻撃が行われており攻撃を検知している

## 検知と調査、対応

- S3バケットの構成変更への調査/対応
    - アカウントのブロックパブリックアクセス設定で、パブリックアクセスをすべてプロックを有効にしていたため、何も変更されていない
    - 手動でパブリックアクセスを有効にすることで検出できた
    - 業務でも有効にできるアカウントは変更していったほうが良さそう

- EC2インスタンス侵害への調査/対応
    - SSH認証方式の確認
        - Inspectorで検出されているはずだけど特に通知なし
        - CloudWatchlogsで`/var/log/secure`ログを確認
            - フィルタリング どちらも特に確認できず
                - [Mon, day, timestamp, ip, id, msg1= Invalid, mag2= user, ...]
                - [Mon, day, timestamp, ip, id, msg1= Accepted, mag2= password, ...]

- IAM認証情報の侵害への調査
    - EC2インスタンスに入られたことでIAMロールが悪用された可能性がある
        - Finding type: UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom
            - 詳細の概要から影響のあったリソースID（IAMロールID)が確認できる
        - 対応
            - IAMロールのアクティブなセッションを無効化
            - EC2インスタンスを停止・起動してアクセスキーをローテーション
                - キーがローテションで来たことでSession ManagerでEC2に接続きた。IAMロールではDenyポリシーがついているのになぜ接続できるのか理解できない...
                - キーの確認 EC2インスタンスに接続後
                    - curl http://169.254.169.254/latest/meta-data/iam/security-credentials/threat-detection-wksp-compromised-ec2
                    - Access Key が変更されていることを確認できた

- Detectiveを使った調査
    - GuardDutyを有効にしたあとに48時間経っていないとだめ
    - 上記を満たしていない状態で有効にしてみたけれども何も表示されなかった
    - CloudTrailのログを検索することでも同様の確認ができるが、視覚的に簡単に確認することができるのが魅了なのだろう

## リソースの削除

- Inspectorのオブジェクトの削除
    - 評価ターゲットの削除
        - 名前: threat-detection-wksp-*

- ロールの削除
    - ロール名: threat-detection-wksp-com-promised-ec2

- Inspector Classic の評価ターゲットの削除

- s3バケットを完全に空にして、削除
    - threat-detection-wksp-data
    - threat-detection-wksp-threatlist
    - threat-detection-wksp-logs

- CloudFormationスタックの削除
    - ThreatDetectionWksp-Attacks
    - ThreatDetectionWksp-Env-Setup

- GuardDutyの無効化

- Security Hubの無効化

- EventBridgeルールの削除
    - post_snstopic_configrule

- CloudWatchLogsの削除
    - /aws/lambda/threat-detection-wksp-additional-configuration
    - /aws/lambda/threat-detection-wksp-remediation-inspector
    - /aws/lambda/threat-detection-wksp-remediation-nacl
    - /threat-detection-wksp/var/log/secure

- Configの無効化

- Detectiveの無効化

