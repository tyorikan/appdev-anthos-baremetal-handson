# ACM Policy Controller によるポリシー制御

<walkthrough-watcher-constant key="region" value="asia-northeast1"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="zone" value="asia-northeast1-c"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="sa" value="sa-baremetal"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="cluster" value="baremetal-trial"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="vm-workst" value="workstation"></walkthrough-watcher-constant>
<walkthrough-watcher-constant key="repo" value="anthos-sample-app"></walkthrough-watcher-constant>

## 始めましょう

[Anthos clusters on bare metal](https://cloud.google.com/anthos/clusters/docs/bare-metal?hl=ja) 上で [ACM Policy Controller](https://cloud.google.com/anthos-config-management/docs/concepts/policy-controller?hl=ja) によるポリシー制御を体験する手順です。

**所要時間**: 約 10 分

**前提条件**:

- Anthos clusters on bare metal クラスタが起動していること
- プロジェクトのオーナー権限をもつアカウントで Cloud Shell にログインしている

**[開始]** ボタンをクリックして次のステップに進みます。

## Policy Controller のインストール

gcloud のデフォルト プロジェクトを設定します。

```bash
export GOOGLE_CLOUD_PROJECT="{{project-id}}"
export logined_account=$(gcloud config get-value core/account)
```

Anthos Config Management のリソース定義を以下のように更新します。

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

`Policy_Controller` が `INSTALLED` になるまで待ちます。10 分弱かかります。

```bash
watch -n 10 gcloud beta container hub config-management status
```

## Policy Controller でのポリシー制御

ACM Policy Controller は [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)をベースにしたポリシー制御機構です。基本的には[クラスタ側のアドミッション制御としてポリシーの準拠を支援](https://cloud.google.com/anthos-config-management/docs/how-to/creating-constraintshl=ja)しますが、CI パイプラインで [ポリシーに反するリソース定義がないかを検証](https://cloud.google.com/anthos-config-management/docs/tutorials/policy-agent-ci-pipeline?hl=ja)したり、[アプリケーションに必須フィールドを要求](https://cloud.google.com/anthos-config-management/docs/tutorials/app-policy-validation-ci-pipeline?hl=ja)したりすることもできます。

ここでは、Kubernetes v1.21 から非推奨となる Pod Security Policy (PSP) に代わり、OPA Gatekeeper ベースのポリシーで特権付きコンテナの実行を防いでみます。

```text
kubectl apply -f - <<EOF 
kind: K8sPSPPrivilegedContainer
apiVersion: constraints.gatekeeper.sh/v1beta1
metadata:
  name: psp-privileged-container
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system"]
EOF
```

`privileged` フラグがついたコンテナを起動しようとすると、Admission 制御で弾かれます。

```bash
kubectl run privileged-test --image=alpine --privileged --restart=Never --command -- cat /etc/os-release
```

## ポリシー違反時にブロックではなく監視のみにする

積極的にブロックするのではなく、ルール違反を監視するだけにしたい場合は `dryrun` 属性を有効化します。

```text
kubectl apply -f - <<EOF 
kind: K8sPSPPrivilegedContainer
apiVersion: constraints.gatekeeper.sh/v1beta1
metadata:
  name: psp-privileged-container
spec:
  enforcementAction: dryrun
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces: ["kube-system"]
EOF
```

そうすると先程のコマンドが通り、Pod が作成されます。

```bash
kubectl run privileged-test --image=alpine --privileged --restart=Never --command -- cat /etc/os-release
```

しかしその違反は Constraint オブジェクトに `violations` として履歴に残ります。

```bash
kubectl get K8sPSPPrivilegedContainer psp-privileged-container -o yaml | grep -B 4 -A 1 'name: privileged-test'
```

## これで終わりです

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

すべて完了しました。リソースの削除にお進みください。

```bash
teachme ~/cloudshell_open/appdev-anthos-baremetal-handson/anthos-baremetal/09-teardown.md
```
