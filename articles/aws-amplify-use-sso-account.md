---
title: "AWS AmplifyでSSOアカウントを利用する"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, amplify]
published: false
---

AWS AmplifyでAWS SSOアカウントを利用するやり方を紹介します

## 事前準備
- AWS OrganisationsおよびSSOを有効化し、AdministratorAccess権限でアクセスできるようにしておく
- amplify/cliのインストール
`npm install -g @aws-amplify/cli`

## aws-sso-utilをインストールする

https://github.com/benkehoe/aws-sso-util#adding-aws-sso-support-to-aws-sdks

```bash
$ brew install pipx
$ pipx ensurepath
$ pipx install aws-sso-util
```

## aws-sso-profileの設定

```bash
# aws configにssoアカウントを設定する(事前にSSOアカウントがconfigに一つはないとエラーになる?)
$ aws configure sso
```

:::details ssoアカウントがconfigになくて発生したエラー
:::message
aws-sso-util configure profile sso-test
Traceback (most recent call last):
  File "/Users/KFMBA/.local/bin/aws-sso-util", line 8, in <module>
    sys.exit(cli())
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 829, in __call__
    return self.main(*args, **kwargs)
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 782, in main
    rv = self.invoke(ctx)
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 1259, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 1259, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 1066, in invoke
    return ctx.invoke(self.callback, **ctx.params)
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/click/core.py", line 610, in invoke
    return callback(*args, **kwargs)
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/aws_sso_util/configure_profile.py", line 88, in configure_profile
    instance = get_instance(
  File "/Users/KFMBA/.local/pipx/venvs/aws-sso-util/lib/python3.9/site-packages/aws_sso_util/utils.py", line 74, in get_instance
    return instances[0]
IndexError: list index out of range
:::


```bash
# sso-testという名称でプロファイルを設定する
$ aws-sso-util configure profile sso-test
SSO start URL [https://d-xxxxxxxxx.awsapps.com/start]:
SSO Region [ap-northeast-1]:
The only AWS account available to you is: xxxxxxxxxxx
Using the account ID xxxxxxxxxxxx
The only role available to you is: AdministratorAccess
Using the role name "AdministratorAccess"
CLI default client Region [None]: ap-northeast-1
CLI default output format [None]: json

To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile sso-test

# configに設定されていることを確認(profileが作成され、credential_processが設定されている)
$ cat ~/.aws/config
[profile sso-test]
sso_start_url = https://d-xxxxxxxxxx.awsapps.com/start
sso_region = ap-northeast-1
credential_process = aws-sso-util credential-process --profile sso-test
sso_account_id = xxxxxxxxxxxxx
sso_role_name = AdministratorAccess
region = ap-northeast-1
output = json
```

## Amplifyで利用する

```bash
# aws ssoで作成したプロファイルを指定してログインする
$ aws sso login --profile sso-test
# AWS_PROFILE環境変数に作成したプロファイルを設定する
$ AWS_PROFILE=sso-test
# Amplifyコマンドを実行
$ amplify -v
Initializing new Amplify CLI version...
Done initializing new version.
Scanning for plugins...
Plugin scan successful
4.43.0
$ amplify init
```

## 参考
- [AWS シングルサインオン を使用するための AWS CLI の設定](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-sso.html)

- https://github.com/aws-amplify/amplify-cli/issues/4488