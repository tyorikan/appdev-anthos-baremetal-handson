# ACM Policy Controller によるポリシー制御

<walkthrough-watcher-constant key="region" value="asia-northeast1"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="zone" value="asia-northeast1-c"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="sa" value="sa-baremetal"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="cluster" value="baremetal-trial"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="vm-workst" value="workstation"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="repo" value="anthos-sample-app"></walkthrough-watcher-constant>

## 始めましょう

[Anthos clusters on bare metal](https://cloud.google.com/anthos/clusters/docs/bare-metal?hl=ja) 上で [ACM Policy Controller](https://cloud.google.com/anthos-config-management/docs/concepts/policy-controller?hl=ja) によるポリシー制御を体験する手順です。

**所要時間**: 約 20 分

**前提条件**:

- Anthos clusters on bare metal クラスタが起動していること
- プロジェクトのオーナー権限をもつアカウントで Cloud Shell にログインしている

**[開始]** ボタンをクリックして次のステップに進みます。

## Anthos Config Management の構成更新

Config Management リソースを以下のように更新します。

```text
cd $HOME
cat << EOF > config-management-spec.yaml
applySpecVersion: 1
spec: 
  policyController:
    enabled: true
    templateLibraryInstalled: true
  configSync:
    enabled: true
    sourceFormat: unstructured
    syncRepo: ssh://${logined_account}@source.developers.google.com:2022/p/${GOOGLE_CLOUD_PROJECT}/r/{{repo}}
    syncBranch: main
    secretType: ssh
EOF
gcloud beta container hub config-management apply \
    --membership="{{cluster}}" --config=config-management-spec.yaml
```

`Status` が `SYNCED` になるまで待ちます。

```bash
watch -n 5 gcloud beta container hub config-management status
```

## これで終わりです

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

すべて完了しました。リソースの削除にお進みください。

```bash
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/09-teardown.md
```
