# ACM Config Sync による設定の同期

<walkthrough-watcher-constant key="region" value="asia-northeast1"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="zone" value="asia-northeast1-c"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="sa" value="sa-baremetal"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="cluster" value="baremetal-trial"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="vm-workst" value="workstation"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="repo" value="anthos-sample-app"></walkthrough-watcher-constant>

## 始めましょう

[Anthos clusters on bare metal](https://cloud.google.com/anthos/clusters/docs/bare-metal?hl=ja) 上で [ACM Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview?hl=ja) による設定の同期を体験する手順です。

**所要時間**: 約 20 分

**前提条件**:

- Anthos clusters on bare metal クラスタが起動していること
- プロジェクトのオーナー権限をもつアカウントで Cloud Shell にログインしている

**[開始]** ボタンをクリックして次のステップに進みます。

## API の有効化

gcloud のデフォルト プロジェクトを設定します。

```bash
export GOOGLE_CLOUD_PROJECT="{{project-id}}"
gcloud config set project "${GOOGLE_CLOUD_PROJECT}"
```

以降のハンズオンで利用する機能を事前に有効化します。

```bash
gcloud services enable connectgateway.googleapis.com sourcerepo.googleapis.com
```

## Cloud Source Repositories の準備

ソースコードを配置するための Git リポジトリを Cloud Source Repository（CSR）に作成します。

```bash
gcloud source repos create {{repo}}
```

実際に作られたリポジトリを確認してみましょう。**Source Repositories** セクションを選びます。

<walkthrough-menu-navigation sectionId="CLOUDDEV_SECTION"></walkthrough-menu-navigation>

## CSR へ接続するための SSH 鍵生成

SSH 認証鍵ペアを作成します。

```bash
ssh-keygen -t rsa -b 4096 -C "$(whoami)" -N "" -f "$HOME/.ssh/csr_rsa"
cat "$HOME/.ssh/csr_rsa.pub"
```

ブラウザから、生成した公開鍵をアップロードします。キー名は任意、鍵は直前で出力された結果のうち **ssh-rsa** で始まる文字列をコピーしてください。

[https://source.cloud.google.com/user/ssh_keys?register=true](https://source.cloud.google.com/user/ssh_keys?register=true)

CSR への接続方法を SSH の設定として保存し

```text
logined_account=$(gcloud config get-value core/account)
cat << EOF >~/.ssh/config
Host source.developers.google.com
    Port 2022
    User ${logined_account}
    IdentityFile $HOME/.ssh/csr_rsa
EOF
```

git クライアントの設定もしておきましょう。

```bash
git config --global user.name "$(whoami)"
git config --global user.email "${logined_account}"
```

##  同期予定のリソース作成

Anthos クラスタ全体で同期したい設定をマニフェストとして用意し

```text
mkdir -p $HOME/{{repo}}/hello
cd $HOME/{{repo}}
cat << EOF > hello/namespace.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: hello
EOF
cat << EOF > hello/color-config.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: color-config
  namespace: hello
data:
  color: purple
EOF
cat << EOF > hello/quota.yaml
kind: ResourceQuota
apiVersion: v1
metadata:
  name: requests-cpu
  namespace: hello
spec:
  hard:
    cpu: "250m"
    memory: 64Mi
EOF
```

リポジトリへ push します。

```bash
git init
git remote add google "https://source.developers.google.com/p/${GOOGLE_CLOUD_PROJECT}/r/{{repo}}"
git checkout -b main
git add .
git commit -m 'init'
git push google main
```

## Anthos Config Management (ACM) の有効化

### ACM の有効化

ACM を初めて使用する場合、次の手順で機能を有効にします。

```bash
gcloud beta container hub config-management enable
```

### ACM Operator のデプロイ

[Connect Gateway](https://cloud.google.com/anthos/multicluster-management/gateway?hl=ja) を通じてクラスタへ接続するためのクレデンシャルをダウンロードし、API エンドポイントが Google Cloud を経由する様子を確認します。

```bash
gcloud container hub memberships get-credentials "{{cluster}}"
kubectl cluster-info
```

Operator をクラスタへデプロイし、インストールされたことを確認しましょう。

```bash
cd $HOME
gsutil cp gs://config-management-release/released/latest/config-management-operator.yaml config-management-operator.yaml
kubectl apply -f config-management-operator.yaml
kubectl -n kube-system get pods | grep config-management
```

リポジトリへアクセスするための秘密鍵を Secret に登録します。

```bash
kubectl create secret generic git-creds --namespace=config-management-system --from-file=ssh=$HOME/.ssh/csr_rsa
```

## Anthos Config Management の構成

Config Management リソースを作成します。

```text
cat << EOF > config-management-spec.yaml
applySpecVersion: 1
spec:
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

リソースが同期されたことを確認します。

```bash
kubectl get ns | grep hello
kubectl get quota -n hello
kubectl get cm -n hello
```

## これで終わりです

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

すべて完了しました。リソースの削除にお進みください。

```bash
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/09-teardown.md
```
