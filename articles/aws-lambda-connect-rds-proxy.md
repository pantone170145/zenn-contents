---
title: "SQLAlchemyでRDS ProxyをIAM認証で接続する際に発生したエラーの解決"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, lambda, rds, postgresql]
published: true
---

## 概要
- AWS LambdaからSQLAlchemyを使ってRDS ProxyをIAM認証で利用しPostgreSQLに接続する際に、以下のエラーに悩まされたので、その解決方法を紹介します
- エラー内容
  :::message alert
  The IAM authentication failed for the role role_name. Check the IAM token for this role and try again.
  :::
  - [RDS Proxy のトラブルシューティング](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/rds-proxy.troubleshooting.html)

## 結論

- `URLエンコード`を利用して、generate_db_auth_tokenで取得したトークンをエンコードする必要がありました

## 詳細

- 実装内容と説明となります

```python
from boto3
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from urllib.parse import quote

try:
  rds = boto3.client('rds')
  region = os.getenv("AWS_REGION")
  cert_path = './cert/AmazonRootCA1.pem'

  password = rds.generate_db_auth_token(
      DBHostname=db_host,
      Port=db_port,
      DBUsername=db_user,
      Region=region
  )
  # URLエンコードが必要だった
  password = quote(password)
  
  url = f"postgresql+psycopg2://{db_user}:{password}@{db_host}:{db_port}/{db_name}?sslmode=verify-full&sslrootcert={cert_path}"
  engine = create_engine(url)
  session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
  # queryなど
  session.execute("SELECT * FROM table_name")
except Exception as e:
  print(f"Exception: {e}")
  raise e
```

### 原因と考察

- ```rds.generate_db_auth_token```した結果に特殊文字が含まれていること
- SQLAlchemyが接続情報を連結してURLとして扱うこと
- これらが原因で不正なトークンとして認識されていたようです
- 他のパッケージは、URLではなくDBの接続情報をそれぞれのパラメータとして渡すことが多いため、このようなエラーは発生しづらいのだと考えられます


## 参考

- 以下、参考にさせていただきました

  - https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/rds-proxy.troubleshooting.html
  - https://qiita.com/akkino_D-En/items/7f39dac2969e120ce695#lambda%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%89%E5%AE%9F%E8%A3%85
    > RDS Proxyを使用する場合はAmazon root CA 1 trust storeが必要となるので以下のURLから取得します。
    > https://www.amazontrust.com/repository/AmazonRootCA1.pem
