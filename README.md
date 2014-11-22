Installing_wordpress_in_ansible
===============================

wordpress4.0日本語版をansibleで導入するplaybookです。
# 1.ディレクトリ構成
```
├── README.md                →　このファイルです。
├── group_vars
│   └── ALL
│       └── mysql_password   →  MySQLに設定する、rootのパスワード、wordpress用のdb名、ユーザー名、パスワードを記載します。
├── hosts.lst                →  対象のサーバー名のFQDNもしくはIPアドレスを記載します。　
├── roles
│   ├── chkconfig
│   │   └── tasks
│   │       ├── chkconfig.yml →  不要サービスを停止するplaybookです。環境にあわせて修正が必要です。
│   │       └── main.yml
│   ├── mysql
│   │   ├── files
│   │   │   └── my.cnf        →  サーバー上の/etc/my.cnfに上書き配布するファイルです。 
│   │   ├── tasks
│   │   │   ├── main.yml
│   │   │   └── mysql.yml     →  MySQLインストール、自動起動設定、rootパスワード変更、wordpress用db作成など行うplaybookです。
│   │   └── templates
│   │       └── my.cnf.j2     →  /root/.my.cnfを作成するにあたっての雛形です。
│   ├── nginx
│   │   ├── files
│   │   │   └── default.conf  →  サーバー上のdefault.confに上書き配布するファイルです。
│   │   └── tasks
│   │       ├── main.yml
│   │       ├── nginx.yml  　 →　nginxインストール、自動起動設定、default.conf上書き配布など行うplaybookです。
│   │       └── php.yml  　 　→　php、関連モジュールインストール、php-fpm自動起動設定など行うplaybookです。
│   └── wordpress
│       └── tasks
│           ├── main.yml
│           └── wordpress.yml →　wordpressダウンロード、config書き換えなど行うplaybookです。
└── site.yml                  →  各playbookを呼び出すYAMLです。（ansible-playbook -i hosts.lst site.yml）
```
# 2.利用方法
1. group_vars/ALL/mysql_password内の、ご自身の環境に合わせて設定したい内容に適宜修正します。
2. roles/chkconfig/task/chkconfig.ymlを、ご自身の環境に合わせて停止したいサービスを記載します。
3. hosts.lstを、ご自身の環境に合わせて、接続したいFQDNもしくはIPアドレスを記載します。
4. site.ymlを、ご自身の環境に合わせて、接続可能なユーザー名に変更します。
5. site.ymlが置かれておりディレクトリまで移動し、以下のコマンドを実行します。
```
$ ansible-playbook -i hosts.lst site.yml
```
# 3.必須修正ファイルについて
group_vars/ALL/mysql_password
```
# mysql上のrootユーザのパスワードを設定する。
mysql_root_password: hoge_root_hoge

# mysql上のwordpressで利用するためのdb名、ユーザー名、パスワードを設定する。
mysql_wordpress_dbname: wordpress
mysql_wordpress_dbuser: wordpress
mysql_wordpress_password: hoge_wordpress_hoge
```

roles/chkconfig/task/chkconfig.yml
```
- name: "common chkconfig( all server )"
  service: name={{ item.name }} enabled={{ item.chkconfig }} state={{ item.s }}
  with_items:
    - { name: 'acpid', chkconfig: 'no', s: 'stopped' }
    - { name: 'irqbalance', chkconfig: 'no', s: 'stopped' }
    - { name: 'postfix', chkconfig: 'no', s: 'stopped' }
    - { name: 'ip6tables', chkconfig: 'no', s: 'stopped' }
# with_items以降の部分を適宜修正する。
```

hosts.lst
```
[PROD]
owlcamp.jp
#wordpressを導入したいサーバー名をFQDNもしくはIPアドレスで指定する。
```

site.yml
```
- hosts: PROD
  #接続したいユーザー名に変更する。
  user: hogehoge111
  sudo: yes
  roles:
      - roles/chkconfig
      - roles/nginx
      - roles/mysql
      - roles/wordpress
```

# 3.site.ymlでの実行rolesの順番について
1. chkconfig
2. nginx
3. mysql
4. wordpress




