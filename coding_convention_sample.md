# Ansibleコーディング規約

## 1. はじめに

Ansibleを開発していくにあたり、「誰が書いても」「誰が見ても」「誰が使っても」わかりやすいものである必要があります、その為にはチームメンバー全員が統一された規約に準拠することが重要です。

ここではAnsibleを書く人、レビューする人、使う人がそれぞれが同じcommon senceを理解しておく為のドキュメンテーションになっています。

また、ここで決められた規約は新しい書き方が必要になった場合に更新されていくこともあります、その為改版されたらその度規約のレビューを実施するプロセスをきちんと実施するなどをしていくことで、規約を作っておしまいではなく、周知/更新していくことも重要です。

## 2. コーディング以外の規約
### ソースコード管理

Ansibleのplaybookを管理するためのツール選定です、本環境ではGitを利用して管理することとします。

#### ソースコード管理ツール

- **Github/Gitlabを利用してください**

### ファイルの拡張子

yamlフォーマットでは.yamlと.ymlの2パターンの書き方をよくみられます、Ansibleでは慣習的に.ymlを利用されているので、本プロジェクトでは.ymlを利用することとします。

#### ファイル拡張子

- **myrole.ymlのように.ymlに統一してください**

#### レポジトリの管理方法

- **role は role 毎に1つのレポジトリを作成してください**
- playbook は1つの目的毎に1つのレポジトリをレポジトリを作成してください。※ただし、切り戻しなど関連する作業は同じレポジトリに入れても良い。
- playbook のレポジトリには複数の playbook を入れても良い
- playbook のレポジトリでは、最上位の階層に playbook ファイルを置いてください。これはサブディレクトリのplaybookを実行すると、role 等の検索パスが変わるためです。

### Ansible versionについて

AnsibleのバージョンはTowerにEEとして設定されている以下とします。

- **ansible-2.9.9 or later**

TowerへのAnsibleインストールにはContainerを作成し登録することとします。

ローカルのAnsibleインストールにはpip3を利用してインストールすることとします。

インストールコマンドは以下を利用してください。

```
pip3 install "ansible==2.9.9"
```

## 3. コーディング規約
### 文字コード

本プロジェクトではLinuxでの標準文字コードである**UTF-8**を利用することとします。

### コマンドを実行するモジュール

command/shell/script/raw の利用は不可避な場合を除いて極力避けるようにしてください。

利用としては冪等性を担保することが難しくなることや、可読性が悪くなることが多くなることが多いからです。

command/shell/script/raw モジュールを安易に使用する前に、目的のモジュールがないか必ず確認してから利用してください。

モジュールインデックスの利用 https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

### Playの書き方

Playは原則、以下のルールに沿って記述してください。

- Play内でroleを呼び出す場合は`roles:`記法を利用してください


### Taskの書き方

taskは原則、以下のルールに沿って記述してください。

- roles/tasks/main.ymlでは他のyamlを**include/importするだけのtask**として作成してください。
- roles/tasks/配下にはそれぞれ役割ごとにymlファイルを分割してください。

また、taskの分割を実施する場合は可能な限り**import_tasks module**を利用して下さい。

もしも、checkだけを行う処理を実現したい場合は**checkだけを行うplayを記述**してください

### Tagsの利用

tagsで処理を制御するユースケースが想定されておらず、tagsを考慮することに寄る可読性低下が懸念されるため、tagsは利用不可とします。

### Inventoryについて

本プロジェクトではStatic Inventoryを利用することとし、Inventoryは各環境(クラスター)単位で分割します。

Inventoryの中ではGroupを「Controller」、「Compute」などの種別毎に記述します。

また、変数は出来るだけ**host_vars, group_vars**は**Inventoryの中に記述する**ようにしてください。これはvarsのファイルが分割されることで変更漏れなどを防ぐためです。

Inventoryのサンプルは以下になります。

```
[controller01.local]
dead:beef::1001 sku=2 bmc_ip=c0de:cafe::2f08

[controller02.local]
dead:beef::1002 sku=2 bmc_ip=c0de:cafe::d4f7

[controller03.local]
dead:beef::1003 sku=4 bmc_ip=c0de:cafe::b81f

[compute01.local]
dead:beef::1401 sku=1 bmc_ip=c0de:cafe::4757

[compute02.local]
dead:beef::1402 sku=1 bmc_ip=c0de:cafe::b0b6

[compute03.local]
dead:beef::1403 sku=1 bmc_ip=c0de:cafe::b115

[cont]
dead:beef::1001 sku=2
dead:beef::1002 sku=2
dead:beef::1003 sku=4

[comp]
dead:beef::1001 sku=2
dead:beef::1002 sku=2
dead:beef::1003 sku=4
dead:beef::1401 sku=1
dead:beef::1402 sku=1
dead:beef::1403 sku=1

[mgmt]
dead:beef::0004 sku=3

[backup]
dead:fee1::1602 region=Tokyo
dead:fee1::1602 region=Osaka

[syslog]
dead:fee1::8001


[backup:vars]
ansible_ssh_user=root
ansible_ssh_pass=root

[mgmt:vars]
ansible_ssh_user=root
ansible_ssh_pass=pass
ansible_sudo_pass=root

[service:children]
cont
comp

[iaas:children]
cont
comp
mgmt

```

### failed_when vs assert

サーバーから取得してきた値が正しいかをヴァリデートする際に、assert、failed_whenのどちらを使っても意図した挙動をさせることができます。

本プロジェクトでは可読性の観点から値のヴァリデートには**assert**を利用することを推奨します。

### Tabとスペース

Tabを使うかスペースを使うかはしばしばプログラムを書く上で議論されています。

一般的にyamlフォーマットで書くときは**スペース(字下げ幅2)**を利用されるので、本プロジェクトでも踏襲します。

### 理論値リテラル(yamllint)

Ansible では様々な記法が許されるtrue/false, True/False, 0/1 …

本プロジェクトではtrue/falseに統一することとします。

#### 理論値リテラル:

-  **true/false (引用符なし)を利用してください。**

### 文字列リテラルの引用符

YAML の文法的には基本的にほとんど文字列リテラルであるため不要なケースも多いが、明確化のため引用符をつけることを推奨されています。

White CloudのStyle guideを踏襲し、本プロジェクトではシングルクォートを利用します。

`https://github.com/whitecloud/ansible-styleguide`

また、シングルクォートを使うことはUnixのベストプラクティスでもあります。

#### 文字列リテラルの引用符ルール

- **シングルクォートを基本的には利用してください**

例外も含めて記載すると下記のようになります。

例外:

- 数字/boolean型
- jinja2のマッピングを利用してシングルクォートの中でネストされている場合
- エスケープ文字列を取り扱いたい場合
- with_itemなどで取り扱うList変数
- スカラー値を使った場合

### キーバリューペア

AnsibleのYAMLパーザでは”key=value”形式を理解する拡張が加えられていますが、YAML標準シンタックスではないので本プロジェクトではリスト記法を利用します。

ex.

##### 悪い例

```
	- name: Ensure mysql is installed
	  yum: name=mysql state=present update_cache=true
```

##### 良い例

```
	- name: Ensure mysql is installed
	  yum:
	    name: mysql
	    state: present
	    update_cache: true
```

### タスク内での宣言の記述順序

宣言順を統一することとより、読みやすさや変更の際に誤りを排除する効果などが期待できます。

本プロジェクトでは下記のように宣言を書いてください。

- task optionはアルファベット順で記述する

ただし、以下のtask optionだけは纏まって利用するので例外とします。

- until、retries、delayはアルファベット順になっていなくても良いのでまとめて記述する

Ex.

```
- name: タスク名
  tags: タグ
  モジュール名:
    # モジュールパラメータ(アルファベット順)
    キー1: 値1
    キー2: 値2
  # ループオペレータ
  with_items:
  - アイテム1
  - アイテム2
  # タスクオプション(アルファベット順): become, ignore_errors, register, when等
  become: true
  register: result
  when: ansible_os_family==’RedHat’
  until: result.stdout|int <= 0
  retries: 10
  delay: 2
```

### 変数の設計

Ansibleでは様々なファイル、手段を用いて変数を宣言することができます。

しかし、全てを自由に使ってしまうと、どこで変数が与えられているかわかりづらくなってしまいます。

そのため、本プロジェクトは下記のソースからしか変数を与えられないよう規約で制限することとします。

- role の default 変数
- inventory ファイルに変数を記載
- extra vars

### ロールの作成方法と命名規則

ロールの作成にはansible-galaxyコマンドを実行します。

**Projectのrootディレクトリ配下**でansible-galaxyコマンドを利用して作成してください。

また、Roleの下にある利用していないディレクトリは削除してください。

EX.

- meta
- handlerなどなど

### Tasknameの命名規則

こちらも変数名の命名規則と同じようにsnake_caseを利用してください。

Roleの命名規則同様に原則、**動詞_名詞**となるように命名してください、また先頭は**大文字**で明記してください。