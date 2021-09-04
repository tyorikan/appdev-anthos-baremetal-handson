# Anthos clusters on bare metal ハンズオン

## v1.8 の構築と ACM によるクラスタ運用

このチュートリアルでは以下を体験できます。

- [Anthos clusters on bare metal](https://cloud.google.com/anthos/clusters/docs/bare-metal/1.8/concepts/about-bare-metal?hl=ja) クラスタの構築
- [ACM Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview?hl=ja) による設定の同期

1. 以下をクリックし、Cloud Shell 環境を起動してください。

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/home/dashboard?cloudshell=true)

2. 以下のコマンドをブラウザ上のターミナルで実行してください。チュートリアルが開始します。

```sh
cloudshell_open --page "shell" \
    --repo_url "https://github.com/google-cloud-japan/appdev-anthos-baremetal-handson.git" \
    --tutorial "appmod/anthos-baremetal/01-setup.md"
```

3. Cloud Shell の再起動や予期せずチュートリアルが消えてしまった場合は、それぞれ以下で再開できます。

```sh
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/01-setup.md
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/02-configsync.md
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/09-teardown.md
```
