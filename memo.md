# terraformerためしたときのメモ

## 試した環境

* PC: MacBook Pro (Retina, 13-inch, Early 2015)
* OS: macOS Big Sur 11.4
* terraformer v0.8.14


## インストール

[installation](https://github.com/GoogleCloudPlatform/terraformer#installation)を見ながらする。

goで作られているらしく、`git clone`して、ビルドする。

```zsh
vaivailx@MacBook-Pro-2 terraformer % git clone https://github.com/GoogleCloudPlatform/terraformer
Cloning into 'terraformer'...

...

Resolving deltas: 100% (20032/20032), done.
vaivailx@MacBook-Pro-2 terraformer % cd terraformer
vaivailx@MacBook-Pro-2 terraformer % go mod download
vaivailx@MacBook-Pro-2 terraformer % go build -v
github.com/aliyun/alibaba-cloud-sdk-go/sdk/auth/credentials

...

github.com/GoogleCloudPlatform/terraformer
vaivailx@MacBook-Pro-2 terraformer %
```

10分以上かかった。
プロバイダーを指定してビルドすることもできるらしいので、
最初は使いたいプロバイダーだけビルドすればいいかも。

使いたいプロバイダーの最小限のファイルを作成する。

```versions.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

terraform initを実行

```zsh
vaivailx@MacBook-Pro-2 terraformer % terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 3.0"...
- Installing hashicorp/aws v3.46.0...
- Installed hashicorp/aws v3.46.0 (signed by HashiCorp)

...

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
vaivailx@MacBook-Pro-2 terraformer %
```

とやっていって、ドキュメントを読み進めたら、パッケージマネージャでインストールできるらしい。

* Homebrew: brew install terraformer
* Chocolatey: choco install terraformer

あとはリリースされたバイナリをダウンロードと言う手段もあるとのこと。
Linuxならばリリースされたバイナリダウンロード、それ以外ならパッケージマネージャが便利かな。

brewでもインストールしたら簡単で早かった。

0.8.14が入っている。
```zsh
vaivailx@MacBook-Pro-2 terraformer % terraformer version
Terraformer v0.8.14
vaivailx@MacBook-Pro-2 terraformer %
```

## 実行お試し

まずはプロバイダ指定だけで実行してみる。

```zsh
vaivailx@MacBook-Pro-2 terraformer % terraformer import aws
2021/06/20 15:37:18 required flag(s) "resources" not set
```

resources指定は必須とのこと。
今はIAM以外はとくに設定していないため、IAMを指定して実行する。


```zsh
vaivailx@MacBook-Pro-2 terraformer % terraformer import aws --resources=iam
2021/06/20 15:42:21 aws importing default region
2021/06/20 15:42:24 aws importing... iam
2021/06/20 15:42:25 operation error IAM: ListUsers, https response error StatusCode: 403, RequestID: 04cd4328-7f30-4ca0-9eab-76d602dfc343, api error AccessDenied: User: arn:aws:iam::682816909333:user/keisuke_yamanaka_personal is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::682816909333:user/

...
```

ずっとエラーがで続ける。
タイムアウトは自動でされないの？と思ったが、オプションであった。

```zsh
  -n, --retry-number          number of retries to perform if refresh fails
  -m, --retry-sleep-ms        time in ms to sleep between retries
```

がすくなくともこれはアクセス権限のエラーでは関係しないよう。
以下のオプション指定をしても変わらずエラーが出続けた。

```zsh
vaivailx@MacBook-Pro-2 terraformer % terraformer import aws --resources=iam -n3 -m3
```

あらためて、十分な権限を付与したユーザーのアクセスキーを発行して、実行

```zsh
vaivailx@MacBook-Pro-2 terraformer % terraformer import aws --resources=iam -n3 -m3

...

2021/06/20 20:41:00 Filtered number of resources for service iam: 35
2021/06/20 20:41:00 aws Connecting....
2021/06/20 20:41:00 aws save iam
2021/06/20 20:41:00 aws save tfstate for iam
vaivailx@MacBook-Pro-2 terraformer %
```

generatedディレクトリが作成されて、terraformのファイルも作成されている。

```zsh
vaivailx@MacBook-Pro-2 terraformer % lsd -l ./generated/aws/iam
.rwxr-xr-x vaivailx staff 173 B  Sun Jun 20 20:41:00 2021  iam_group.tf
.rwxr-xr-x vaivailx staff 696 B  Sun Jun 20 20:41:00 2021  iam_group_policy_attachment.tf
.rwxr-xr-x vaivailx staff 188 B  Sun Jun 20 20:41:00 2021  iam_instance_profile.tf
.rwxr-xr-x vaivailx staff 5.9 KB Sun Jun 20 20:41:00 2021  iam_role.tf
.rwxr-xr-x vaivailx staff 3.0 KB Sun Jun 20 20:41:00 2021  iam_role_policy_attachment.tf
.rwxr-xr-x vaivailx staff 591 B  Sun Jun 20 20:41:00 2021  iam_user.tf
.rwxr-xr-x vaivailx staff 305 B  Sun Jun 20 20:41:00 2021  iam_user_group_membership.tf
.rwxr-xr-x vaivailx staff 381 B  Sun Jun 20 20:41:00 2021  iam_user_policy_attachment.tf
.rwxr-xr-x vaivailx staff 6.1 KB Sun Jun 20 20:41:00 2021  outputs.tf
.rwxr-xr-x vaivailx staff 100 B  Sun Jun 20 20:41:00 2021  provider.tf
.rwxr-xr-x vaivailx staff  52 KB Sun Jun 20 20:41:00 2021  terraform.tfstate
vaivailx@MacBook-Pro-2 terraformer %
```

terraformerでimportしたあとにterraform planすると差分がでると聞いていた気がするので試した。

```zsh
vaivailx@MacBook-Pro-2 terraformer % terraform plan

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
vaivailx@MacBook-Pro-2 terraformer %
```

でないね。iamはわりとシンプルに定義できるからだろうか？
