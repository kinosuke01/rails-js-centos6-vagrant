# これはなに？
CentOS6ベースのRails+Javascript開発環境を構築するVargrant設定ファイルです。


# 開発サーバ作成手順

## boxファイルを取得

```
vagrant box add CentOS_6_7 https://github.com/CommanderK5/packer-centos-template/releases/download/0.6.7/vagrant-centos-6.7.box
```

## ディレクトリ作成 (作成場所は適宜読み替えてください)

```
cd ~/vagrant
mkdir golem
cd golem
vagrant init CentOS_6_7
rm Vagrantfile
git clone #{リポジトリURL} .
```


# 起動+プロビジョニング

```
vagrant up
```


# 補足
htmlディレクトリはnginxのDocumentRootに設定しています。
静的ファイルを置いて動作確認などにご利用ください。

