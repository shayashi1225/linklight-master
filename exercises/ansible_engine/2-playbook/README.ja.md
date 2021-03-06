# Exercise 2 - 初めてのplaybook作成


この演習では *playbook* を作成してみましょう。  
playbookは 先程実行した複数の ad-hoc コマンドを、*plays* と *tasks* のセットにし、ファイルへ置き換えたものです。

playbookは1つまたは複数のplayに、1つまたは複数のタスクを持つことができます。  
*play* の目的はホストのグループをマッピングする事であり、*task* の目的はこれらのホストに対してモジュールを実行する事です。

この演習のplaybookでは、1つのplayに2つのtaskを書いていきたいと思います。


## Section 1: ディレクトリとplaybookファイルの作成

Ansibleには、playbookディレクトリ構成の[ベスト・プラクティス](http://docs.ansible.com/ansible/playbooks_best_practices.html)があります。Ansibleのスキルを得て熟練される場合は、こちらを読み、理解いただくことを強くお勧めします。本日のplaybookはとても基本なものですが、入り組んだ構造から少し複雑に映るかも知れないからです。

今回は、とてもシンプルなplaybookディレクトリ構成と、そこに複数ファイルを追加する簡易な形で進めます。


### Step 1:
以下のように *apache_basic* をホームディレクトリとして作成し、作成したディレクトリに移動します。

```bash
mkdir ~/apache_basic
cd ~/apache_basic
```

### Step 2:
 `vi` または `vim` で `install_apache.yml` ファイルを作成編集します。


## Section 2: Play の定義

現在編集中の  `install_apache.yml` に、Playを定義し、各行の内容を理解しましょう。


```yml
---
- hosts: web
  name: Install the apache web service
  become: yes
```

- `---` YAML開始を定義
- `hosts: web` はこのPlaybookを実行する対象のグループを指定しています。グループ名はInvenotryファイルで定義されています。
- `name: Install the apache web service` Playに名称を設定しています。任意の名称設定が可能です。名称を付けず、スキップすることもできます。
- `become: yes` リモートホストでroot権限を使い実行するためのオプションを指定しています。デフォルトはsudoですが、su、pbrun、[その他](http://docs.ansible.com/ansible/become.html) もサポートしています。


## Section 3: Play内のTask追加

ここまででplayを定義しました。  
続けて実行する複数のtaskを追加しましょう。  
このあとに記述する`task` の *t* と、先ほどのplayで記載した`become` の *b* が、インデントで同じ位置に来るように調整してください。  
この記述の仕方がとても重要です。  

Playbook内の記述は、すべてここで示されている形式に倣う必要があります。
この演習で作成するPlaybookの全文を確認したい場合は、この演習ページの最下部を参照してください。


```yml
tasks:
 - name: install apache
   yum:
     name: httpd
     state: present

 - name: start httpd
   service:
     name: httpd
     state: started
```

- `tasks:` 1つまたは複数のtask定義がされることを示す
- `- name:` playbookの実行時に標準出力されるそれぞれのtask名称
よって、出力されるに完結かつ、適切なtask名を記入してください。


```yml
yum:
  name: httpd
  state: present
```


- これらの3行は httpd をインストールするため Ansible の *yum* モジュールを呼び出しています。
yum モジュールの全オプションを確認するには[こちらをクリック](http://docs.ansible.com/ansible/yum_module.html)


```yml
service:
  name: httpd
  state: started
```

- 次の複数行は、httpd サービスを開始するため、*service*  モジュールを呼び出しています。このモジュールはリモートホストのサービスを制御する際使うのに適してます。
service モジュールの全オプションを確認するには[こちらをクリック](http://docs.ansible.com/ansible/service_module.html)


## Section 4: playbookの保存

playbookを書き終えたら、保存しましょう。

`vi` または `vim`にて、`write/quit` を使用(例: Escキー押下後、wq!実行)し、playbookを保存します。

これにより、playbook `install_apache.yml` が完成です。自動化準備OKとなります。

---
**NOTE**
Ansible (実際にはYAML) はインデントやスペースの形式が少し特殊かもしれません。後ほど余計な混乱を来さぬよう [YAML シンタックス](http://docs.ansible.com/ansible/YAMLSyntax.html) を、その際に完成したplaybookを傍らに置きながらご覧いただき、スペースやアラインメントをご確認いただくことをお勧めいたします。


---

```yml
---
- hosts: web
  name: Install the apache web service
  become: yes

  tasks:
    - name: install apache
      yum:
        name: httpd
        state: present

    - name: start httpd
      service:
        name: httpd
        state: started
```

## Section 5: playbookの実行

では3つのWebノードで、このまっさらのPlaybookを走らせてみましょう。そのためにはansible-playbookコマンドを使います。

Playbookのあるディレクトリ（~/apache_basic）からPlaybookを走らせます。

```bash
ansible-playbook install_apache.yml
```
でもこのコマンドを走らせるのを少しだけ待って、ここでは使用していませんが、オプションに目を向けてみましょう。

- *-i* このオプションは、利用する特定のInventoryファイルを指定します。
- *-k* このオプションは、ユーザがPlaybookを走らせる際にパスワードを要求します。
- *-v* このオプションを使えばverboseモードでPlaybookを走らせることができます。2回目にPlaybookを走らせる際、-vまたは-vvを利用して表示されるメッセージを増やしてみてください。
- *--syntax-check* 意図したとおりにPlaybookが走らない場合は、このオプションを使えばどこで問題が発生しているのか調べることができます。コードのコピー／ペーストで発生したによる問題箇所も見つけることができるはずです。
```bash
ansible-playbook install_apache.yml --syntax-check
```


それでは、Playbookを走らせてみましょう。

標準の出力では、以下のような内容が表示されるはずです：
```
[student1@ansible apache_basic]$ ansible-playbook install_apache.yml

PLAY [Install the apache web service] **********************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [node2]
ok: [node3]
ok: [node1]

TASK [install apache] **************************************************************************************************
changed: [node2]
changed: [node3]
changed: [node1]

TASK [start httpd] *****************************************************************************************************
changed: [node3]
changed: [node1]
changed: [node2]

PLAY RECAP *************************************************************************************************************
node1                      : ok=3    changed=2    unreachable=0    failed=0   
node2                      : ok=3    changed=2    unreachable=0    failed=0   
node3                      : ok=3    changed=2    unreachable=0    failed=0   
```
playと各taskには名前がつけれられているため、何が、どのノードに対して行われたのかを把握できます。また、記述していないGathering Factsというtaskがあることにも気がつくはずです。これは、この[setupモジュール](https://docs.ansible.com/ansible/latest/modules/setup_module.html)がデフォルトで走るようになっているからです。これをOFFにしたい場合は、playの中で以下のようにgather_facts: falseと定義します：
```yml
---
- hosts: web
  name: Install the apache web service
  become: yes
  gather_facts: false
```

---
[Ansible Linklightのページへ戻ります - Ansible Engine Workshop](../README.ja.md)


[次のExerciseへ](../3-variables/README.ja.md)
