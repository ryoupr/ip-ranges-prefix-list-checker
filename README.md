# ip-ranges-prefix-list-checker

AWS の [ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json) の更新を検知し、指定したサービス・リージョンの IP レンジに変更があればメールで通知する仕組みです。

Direct Connect 環境で Prefix List を運用している場合に、IP レンジの変更を見逃さないための自動検知ツールとして利用できます。

## アーキテクチャ

SNS (`AmazonIpSpaceChanged`) → Lambda → S3（差分比較） → SNS（メール通知）

## デプロイ

**us-east-1 にデプロイしてください**（AmazonIpSpaceChanged SNS トピックが us-east-1 固定のため）。

```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name ip-ranges-prefix-checker \
  --template-file cfn/ip-ranges-prefix-list-checker.yaml \
  --parameter-overrides NotificationEmail=your@email.com \
  --capabilities CAPABILITY_IAM
```

デプロイ後、通知先メールアドレスに SNS サブスクリプションの確認メールが届きます。`Confirm subscription` リンクをクリックしてください。

## パラメータ

| パラメータ | デフォルト値 | 説明 |
|-----------|-------------|------|
| `NotificationEmail` | （必須） | 変更通知の送信先メールアドレス |
| `TargetServices` | `AMAZON_CONNECT,CLOUDFRONT` | 監視対象の AWS サービス名（カンマ区切り） |
| `TargetRegions` | `ap-northeast-1,GLOBAL` | 監視対象のリージョン（カンマ区切り） |

## クリーンアップ

```bash
BUCKET=$(aws cloudformation describe-stacks \
  --region us-east-1 \
  --stack-name ip-ranges-prefix-checker \
  --query "Stacks[0].Outputs[?OutputKey=='BucketName'].OutputValue" \
  --output text)

aws s3 rm s3://${BUCKET} --recursive
aws cloudformation delete-stack --region us-east-1 --stack-name ip-ranges-prefix-checker
```

## 関連ブログ

TODO: はてなブログ記事のURLを差し替え

## License

MIT
