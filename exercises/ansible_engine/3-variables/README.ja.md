# Exercise 3 - 変数、ループ、ハンドラを使う

先の演習ではAnsible Coreの基本的な部分をご紹介しました。 この後のいくつかの演習では、Playbookに柔軟性を持たせ、そしてパワフルなものへと変えていけるよう、もう一歩踏み込んだ内容について取り上げていきます。

Ansibleの最大の利点は、taskをシンプルかつ再利用だといえます。しかし、全てのシステムが一様に同じではなく、AnsibleのPlaybookを走らせる際に少しばかり変更が必要となることも考えられます。ここで変数が登場します。

システム間の差異を変数で埋め、ポートやIPアドレス、そしてディレクトリの変更が出来るようにします。

ループを使えば同じtaskを何度でも繰り返すことができます。10のパッケージをインストールする場合などが分かりやすい例でしょう。 ループを用いれば、これを1つのtaskで行えます。

ハンドラはサービスの再起動に用います。新しい構成ファイルを用意したり、新しいパッケージをインストールしたら、サービスを再起動してそれら加えた変更が反映されるようにするはずです。これを担うのがハンドラです。

- [Ansible 変数](http://docs.ansible.com/ansible/latest/playbooks_variables.html)
- [Ansible ループ](http://docs.ansible.com/ansible/latest/playbooks_loops.html)
- [Ansible ハンドラ](http://docs.ansible.com/ansible/latest/playbooks_intro.html#handlers-running-operations-on-change)

## Section 1:  Playbookへの変数とループの追加

まず最初に新しいPlaybookを作成しますが、これは先の演習 1.2で作成したものによく似ています。


### Step 1:

ホームディレクトリへ移動し、新しいプロジェクトとPlaybookを作成します。

```bash
cd
mkdir apache-basic-playbook
cd apache-basic-playbook
vim site.yml
```


### Step 2:

playの定義といくつかの変数をPlaybookに追加します。このPlaybook中には、利用しているWebサーバへの追加パッケージのインストールと、Webサーバに特化したいくつかの構成が含まれています。

```yml
---
- hosts: web
  name: This is a play within a playbook
  become: yes
  vars:
    httpd_packages:
      - httpd
      - mod_wsgi
    apache_test_message: This is a test message
    apache_max_keep_alive_requests: 115
```


### Step 3:

*install httpd packages* と命名した新規taskを追加します。

```yml
  tasks:
    - name: install httpd packages
      yum:
        name: "{{ item }}"
        state: present
      with_items: "{{ httpd_packages }}"
      notify: restart apache service
```

---
**NOTE**

- `vars:` この後に続いて記述されるものが変数名であることをAnsibleに伝えています
- `httpd_packages` httpd_packagesと命名したリスト型（list-type）の変数を定義しています。その後に続いているのは、このパッケージのリストです
- `{{ item }}` この記述によって `httpd` や `mod_wsgi` といったリストのアイテムを展開するようAnsibleに伝えています。
- `with_items: "{{ httpd_packages }}` Ansibleに `httpd_packages` の `item` 毎にタスクをループ実行するよう Ansible に伝えます
- `notify: restart apache service` この行は `handler`であり、詳細は Section 3 で触れます

---

## Section 2: ファイルの実装とサービスの起動

ファイルやディレクトリを扱う必要がある場合には、[Ansible ファイル](http://docs.ansible.com/ansible/latest/list_of_files_modules.html) モジュールを用います。
今回は `file` や `template` モジュールを利用します。

その後、Apacheのサービスを起動するtaskを定義します。


### Step 1:
apache-basic-playbookディレクトリの中に`templates`ディレクトリを作成し、2つのファイルをダウンロードします。

```bash
mkdir templates
cd templates
curl -O http://ansible-workshop.redhatgov.io/workshop-files/httpd.conf.j2
curl -O http://ansible-workshop.redhatgov.io/workshop-files/index.html.j2
```

### Step 2:
いくつかの file task と service task をplaybookに追加します。

```yml
- name: create site-enabled directory
  file:
    name: /etc/httpd/conf/sites-enabled
    state: directory

- name: copy httpd.conf
  template:
    src: templates/httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify: restart apache service

- name: copy index.html
  template:
    src: templates/index.html.j2
    dest: /var/www/html/index.html

- name: start httpd
  service:
    name: httpd
    state: started
    enabled: yes
```

---
**NOTE**

- `file:` このモジュールを使ってファイル、ディレクトリ、シンボリックリンクの作成、変更、削除を行います。
- `template:` このモジュールで、jinja2テンプレートの利用と実装を指定しています。 `template` は `Files` モジュール・ファミリの中に含まれています。その他の [file-management modules here](http://docs.ansible.com/ansible/latest/list_of_files_modules.html) についても、一度目を通しておくことをお勧めします。
- `jinja` [jinja2](http://docs.ansible.com/ansible/latest/playbooks_templating.html)は、Ansibleでテンプレートの中のfiltersのような式の中のデータを変更する場合に用います。
- `service` serviceモジュールはサービスの起動、停止、有効化、無効化を行います。

---

## Section 3: ハンドラの定義と利用

構成ファイルの実装や新しいパッケージのインストールなど、様々な理由でサービスやプロセスを再起動する必要が出てきます。このセクションには、Playbookへのハンドラの追加、そして意図しているtaskの後でこのハンドラを呼び出す、という2つの内容が含まれています。それではPlaybookへのハンドラの追加を見てみましょう。

### Step 1:
ハンドラを定義する。

```yml
handlers:
  - name: restart apache service
    service:
      name: httpd
      state: restarted
      enabled: yes
```

---
**NOTE**

- `handler:` これで *play* に対して `tasks:` の定義が終わり、`handlers:` の定義が開始されたことを伝えています。これに続く箇所は、名前の定義、そしてモジュールやそのモジュールのオプションの指定のように他のtaskと変わらないように見えますが、これがハンドラの定義になります。
- `notify: restart apache service` そしてついに、この部分でハンドラが呼び出されるのです！ `nofify` 宣言は名前を使ってハンドラを呼び出します。単純明快ですね。先に書いた `copy httpd.conf` task中に `notify` 宣言を追加した理由がこれで理解できたと思います。

---

## Section 4: この演習の最後に

これで洗練されたPlaybookの完成です!  
でもまだPlaybookを実行しないでください。それはこの後の演習で行います。  
その前に、全てが意図した通りになっているかもう一度見直してみましょう。  
もしも間違っていれば修正してください。  
以下の見本を参考に、スペースとインデントに注意して見てください。

```yml
---
- hosts: web
  name: This is a play within a playbook
  become: yes
  vars:
    httpd_packages:
      - httpd
      - mod_wsgi
    apache_test_message: This is a test message
    apache_max_keep_alive_requests: 115

  tasks:
    - name: httpd packages are present
      yum:
        name: "{{ item }}"
        state: present
      with_items: "{{ httpd_packages }}"
      notify: restart apache service

    - name: site-enabled directory is present
      file:
        name: /etc/httpd/conf/sites-enabled
        state: directory

    - name: latest httpd.conf is present
      template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: restart apache service

    - name: latest index.html is present
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html

    - name: httpd is started and enabled
      service:
        name: httpd
        state: started
        enabled: yes

  handlers:
    - name: restart apache service
      service:
        name: httpd
        state: restarted
        enabled: yes
```

---

[Ansible Linklightのページへ戻ります - Ansible Engine Workshop](../README.ja.md)


[次のExerciseへ](../4-runplaybook/README.ja.md)
