---
title: "The IAM authentication failed for the role role_name.の解決"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, lambda, rds]
published: false
---

## 概要
- AWS LambdaからRDS ProxyをIAM認証で利用しPostgreSQLに接続する際に、以下のエラーが発生したので、その解決方法を紹介します
- エラー内容
    - `The IAM authentication failed for the role role_name. Check the IAM token for this role and try again.`
    - [RDS Proxy のトラブルシューティング](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/rds-proxy.troubleshooting.html)
- SQLAlchemyを利用して、接続する際に発生しました
  - [SQLAlchemy](https://www.sqlalchemy.org/)

## 解決方法
- `urlencode`を利用して、generate_db_auth_tokenで取得したトークンをエンコードする必要がありました

## 