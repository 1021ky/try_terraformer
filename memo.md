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

