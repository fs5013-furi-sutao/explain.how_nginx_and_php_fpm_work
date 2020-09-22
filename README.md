# explain.how_nginx_and_php_fpm_work
## Nginx と PHP-FPM の仕組み
Nginx と PHP-FPM の仕組みについて説明をしていく。

おもに、以下の疑問に答えていく。
1. PHP-FPM って何？
2. Apache の場合インストールしただけで PHP の実行環境構築できるよね？
3. nginx と PHP-FPM の通信はどうなってんの？

### 疑問 1 : PHP-FPM って何？
FPM ( FastCGI Process Manager ) は PHP の FastCGI 実装のひとつで、 主に高負荷のサイトで有用な追加機能を備えている。

#### そもそも CGI って何？
Common Gateway Interface（コモン・ゲートウェイ・インタフェース、CGI）は、 ** ウェブサーバ上でユーザプログラムを動作させるための仕組み。 ** 現存する多くの Web サーバプログラムでは CGI の機能を利用することができる。

Web サーバプログラムの機能の主体は、あらかじめ用意された情報を利用者（クライアント）の要求に応じて送り返すことである。そのためサーバプログラム単体では情報をその場で動的に生成してクライアントに送信するような仕組みを作ることはできなかった。 そこでサーバプログラムから他のプログラムを呼び出し、その処理結果をクライアントに送信する方法が考案された。それを実現するためのサーバプログラムと外部プログラムとの連携法の取り決めが CGI である。

つまり、Web サーバ上で PHP ( 動的コンテンツを生成する言語 ) を動作せるための仕組みである。

#### じゃあ FastCGI は？
FastCGI とは、Web サーバ上でユーザプログラムを動作させるためのインタフェース仕様の一つである。 CGI の問題を解決するために Open Market 社によって1990 年代中頃に開発されたもので、プロトコルが公開されている。

CGI は、ユーザから要求がある度に、プロセスの生成と破棄を行う。 大量の要求があればその分だけプロセスの生成と破棄が実施され、この事がパフォーマンスの悪化に繋がっていく。

FastCGI は、初回リクエスト時に起動したプロセスをメモリ上へ保持を行い、次回リクエストに対してはそのメモリに保持したプロセスの実行を行う。 それまでの CGI の問題を解決し、プログラム動作速度の向上およびサーバ負荷の低下を可能にしたのが FastCGI である。

### 疑問 2 : Apache の場合インストールしただけで PHP の実行環境構築できるよね？
PHP には、モジュール版と CGI 版という二種類がある。

Apache ではモジュールとして PHP スクリプトを起動することができる。これにより、Web サーバの起動時に読み込まれサーバの処理の一部として実行される。 PHP をインストールすると自動的に PHP モジュールもインストールされ、 Apache はデフォルトでその PHP モジュールを読み込む。

Nginx では上記で説明した通り、 FastCGI を通して PHP を実行することができる。

#### モジュール版？
モジュール版は CGI 版の逆で、Web サーバのプロセスのなかで PHP を実行してしまう方法である。 現在多くのレンタルサーバは Web サーバーに Apache を採用しているため、モジュール版というと、通常は Apache のモジュール版を意味する。

ただ PHP がWeb サーバを動かすユーザ（root 権限など）で動作するため、ユーザーが複数いる共用サーバーではセキュリティ面に不安がある。そこでモジュール版には「セーフモード」という設定があり、ユーザー間のファイル干渉を防止できるようなっている。

またモジュール版のメリットとして、Webサーバーのプロセスで PHP が実行されるため、CGI に比べて動作速度が高速になるという点がある。

#### CGI版？
CGI 版は実行ファイル形式とも呼ばれ、Web サーバとは別のプロセスで実行される。このメリットとしては、まずセキュリティ面が挙げられる。CGI 版の PHP を動かす各ユーザは、Web サーバ本体を動かすユーザとは異なる（切り離されている）。そのため誤って他ユーザーに干渉してしまうといった危険がない。

一方デメリットとしては、Web サーバとは別個のプロセスとして動かす分、実行するたびにメモリのロードが必要となり、動作速度がモジュール版に比べて遅くなる。

#### Apache では FastCGI を通してPHPを実行することはできんの？
Apache でも可能。

### 疑問 3 : Nginx と PHP-FPM の通信はどうなってんの？
TCP か UNIX ドメインソケットのどちらかで通信するのがメジャー。

#### TCP と UNIX ドメインソケットって何？違いは？
UNIX ドメインソケットは、TCPソケット(INETドメインソケット)よりも遥かにスループットが優れている。
