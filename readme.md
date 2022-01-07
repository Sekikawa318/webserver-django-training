# 基盤からWebサイトを作る
## 目的
1. 基盤の基礎知識を身に付ける
2. モダンな技術に触れる
3. 自分で考えてAWSで環境を構築することで、学んだ内容を実践する
    - 手順をコピペするだけではなく、自分で考えて基盤周りを組み立てられるようにする

---

## 手順
1. 必要なものをインストール
    - Python
    - Django
    - Nginx
    - uWSGI
2. Djangoでhello Worldサイトを作成する
3. uWSGIサーバの設定
4. Webサーバ(Nginx)の設定

---

## 1. 必要なものをインストール
### 1. Python
- [Pythonインストールサイト](https://www.python.org/downloads/)からインストールする

### 2. Django
```shell
pip install django 
pip show django
```
- djangoの情報がでればOK

### 3. Nginx
#### macの人(homebrewが入っていない人は[インストール](https://brew.sh/index_ja))
```shell
brew install nginx
```

#### windowsの人 ⇒ Windows環境は試していないので出来ればmacかlinuxを使って欲しい
- [サイトからインストール](http://nginx.org/en/download.html)

### 4. uWSGI
```
pip install uwsgi
pip show uswsgi
```
- uwsgiの情報がでればOK

---

## 2. Djangoでhello worldサイトを作成する[参考](https://docs.djangoproject.com/ja/4.0/intro/tutorial01/)
1. プロジェクトを作りたいディレクトリに移動
2. 以下コマンドを実行する

```sh
django-admin startproject helloWorld
```

3. helloWorldディレクトリが作成されるのでhelloWorldディレクトリに移動
4. python manage.py startapp helloWorld
5. helloWorld/settings.pyにhelloWorldを追加する

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'helloWorld',
]
```

6. helloWorld/views.pyを以下のように編集する

```python
from django.http import HttpResponse
 
def helloWorld(request):
    # リクエストに対して、Hello Worldを返す
    return HttpResponse("Hello World")
```

7. helloWorld/urls.pyを以下のように編集する

```python
from django.conf.urls import url
from . import views

urlpatterns = [
    # 「localhost:port/helloWorld」のリクエストを受け取ったときに、views.pyのhelloWorld関数に飛ばす
    url("", views.helloWorld, name='helloWorld'),
]
```
8. プロジェクトディレクトリ直下にあるurls.pyを以下のように編集する

```python
from django.conf.urls import include, url
from django.contrib import admin
 
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    # 「localhost:port/helloWorld」のリクエストを受け取ったときに、helloWorld/urls.pyに飛ばす
    url(r'^helloWorld/', include('helloWorld.urls')),
]
```

9. 試してみる
```sh
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

- 以下にアクセスしてHello Worldが出力されればOK
  - http://localhost:8000/helloWorld/
- 問題なければ、Ctr+Cでrunserverを止める

---

## 3. uWSGIサーバの設定[参考](https://qiita.com/ming_hentech/items/9e21fe175988448e204b)

```sh
# 8000ポートでサーバ起動(uwsgiはWebサーバ機能も持っているらしい)
uwsgi --http :8000 --module helloWorld.wsgi --chdir [プロジェクトのパス]

# uwsgi --http :8000 --module helloWorld.wsgi --chdir ~/aws/nri/django/helloWorld
```

- 以下にアクセスして、Hello Worldが出力されればOK
  - http://localhost:8000/helloWorld/

---

## 4. Webサーバ(Nginx)の設定
1. 以下のコマンドを実行して起動確認する
```sh
nginx
```

2. 以下にアクセスしてwelcome to nginxが表示されればOK
    - http://localhost:8080


3. 「uwsgi_params」ファイルをプロジェクト直下に作成
4. 以下の内容をコピペ

```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;
 
uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;
 
uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

5. 「helloWorld_nginx.conf」ファイルを作成して、以下の内容を記載

```conf
upstream django {
    server 127.0.0.1:8001;
}
 
# configuration of the server
upstream django {
    server 127.0.0.1:8001;
}
 
# configuration of the server
server {
    listen      8000;
    server_name 127.0.0.1;
    charset     utf-8;
 
    location /static {
        alias /Users/username/aws/nri/django/helloWorld/static;
        # alias /Users/sekikawasatoshi/aws/nri/django/helloWorld/static;
    }
 
    location / {
        uwsgi_pass  django;
        include     /Users/username/aws/nri/django/helloWorld/uwsgi_params;
        # include     /Users/sekikawasatoshi/aws/nri/django/helloWorld/uwsgi_params;
    }
}
```

6. シンボリックリンクを作成する

```sh
# nginxはnginx/servers直下にあるconfファイルを見に行く
sudo ln -s [confファイルの場所] [nginxのインストール場所/servers]
# sudo ln -s ~/aws/nri/django/helloWorld/helloWorld_nginx.conf /opt/homebrew/etc/nginx/servers/
```

7. settings.pyの末尾に以下を追記
```python
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

8. 以下のコマンドを実行してstaticディレクトリの作成する


```sh
python manage.py collectstatic
```

9. 以下のコマンドでnginxを起動する

```sh
# 既に起動している場合
nginx -s stop
nginx
```

10. 以下のコマンドでuwsgiを起動する

```sh
uwsgi --socket :8001 --module helloWorld.wsgi --chdir [プロジェクト直下までのパス]
# uwsgi --socket :8001 --module helloWorld.wsgi --chdir ~/aws/nri/django/helloWorld
```

11. 以下のURLにアクセスしてHello Worldが表示されることを確認する
    - http://localhost:8000/helloWorld/
