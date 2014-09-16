isucon3_prepare
===============

あと欲しいチートシート:

- redis/resque ruby コードチートシート

アイデア：

- erb => slim (そんなにキモにはならなそう)
- varnish 使う (これは当然)
- mysql のデータファイルを tmpfs においたらめっちゃはやそう

    - レギュレーションの「サーバの設定およびデータ構造は任意のタイミングでのサーバ再起動に耐えること」にひっかかるのでだめっぽい T T
    - シャットダウン時にディスクにおいて、起動時にディスクから tmpfs に cp するようにすればいいんじゃないのかな
    - 実際にやってみたが速くはなったが、varnish のほうが銀の弾丸度では上だな。アプリの処理を丸っと省けるからな。

- 最初だけアプリに newrelic か rack-mini-profiler 仕込んだほうがわかりやすいかも
- rubinius への置き換え => 使ったことないのでエラー出た時どうしたらいいかわからん
- AMIが提供されるだけなので、インスタンス１つではなく、複数立ちあげてそれぞれで別作業をするのも可。ただ、他に展開するためにAMI保存しようとするとその間なにも作業できなくなるので、手で移植か？

cf. http://blog.nomadscafe.jp/2012/11/isucon2-5.html

- kazeburo さんは worker に HTML を保存させてそれを serve するようにしたらしい https://github.com/kazeburo/isucon2_hack/blob/master/perl/worker.pl
- nginx にアクセス時に gzip 圧縮するのではなく、その HTML をあらかじめ gzip 圧縮して配信したらしい 
- トラッフィク多すぎたので delay を入れたとな ...

cf. http://www.seeds-std.co.jp/seedsblog/163.html

memcached の中に入ってるキーを一覧するperlワンライナー

```
perl -MCache::Memcached -e '$s="localhost:11211";$m=new Cache::Memcached({servers=>[$s]});$res=$m->stats("items");$i=$res->{hosts}{$s}{items};@a=split("€n",$i);while(){if($_=~/items:([0-9]+)/){$s{$1}=$_}};foreach $key (keys %s){$cm="cachedump $key 100";$res=$m->stats($cm);print "--- €n".$cm."€n";print $res->{hosts}{$s}{$cm}}'
```

再起動時にキャッシュに突っ込むシェルスクリプト

```
#!/bin/sh

for i in `seq 1 1 3000`
do
    wget http://app01/article/$i -O -
done
```

デプロイ. てっとりはやく rsync でいいかもね

```
#!/bin/sh
rsync -Irp --exclude ".git" --progress . $HOST:isucon/webapp/ruby/
ssh -t $HOST sudo /etc/init.d/supervisord restart
```

# ISUCON3 レギュレーションと予選当時の流れ

- http://isucon.net/archives/cat_1024990.html
- https://gist.github.com/941/191180fd8e33aae121cb

# 当日の行動

- 予選 10月5日(土) 10:00 - 18:00 => 10月6日(日) 10:00 - 18:00 に変更しました
- 遅くとも 09:30 にはヒカリエに集まる. 席を近くにする(モニタは借りればよい.休みなので使ってない)
- ご飯は食べておく and 飲み物とか買っておく

# 次回に向けて（反省）

- まず初期設定(OK)

  - ISUCONアプリはログが出ないので Logger しこむとか、byebug いれるとかデバグしやすい環境を作っておく
  - デプロイは git pull より cp -a のほうが良かったかも(capistrano はやりすぎ)

- 直せそうな所を見つけてすぐ直すよりも、一旦まとめて戦略を錬ってガッとやる方が上位にいけるっぽい
- 洗い出し(90分ぐらい)。簡単な改修ポイントがみつかってもまだ手を出さず、洗い出しに専念したほうがよさそうか

  - (一人)メトリクスを出す仕込みを入れて一回ベンチ通す
  
    - ベンチを１回走らせるのに何秒かかるか time コマンドをかましてとっておく。
    - エンドポイントごとのtemplate, db メトリクス(呼び出し回数、平均時間、合計時間)を出してとっておく。見ておく
    - query ごとのメトリクスを出しておく(long_query_time = 0 で slow.log 出しておく). mysqlslowdump で解析してとっておく。見ておく
    - lsof してアプリがどこにアクセスしているのかとっておく。見ておく
    - vmstat, iostat の結果をとっておく。見ておく
    - テーブルのスキーマ、インデックス情報などをみておく

  - アプリの仕様、画面とコードの対応箇所(構造)を把握しておく
  - そして、コードを読んで改善できところをチェックしていく(局所的)
  - それとは別に大幅に変更すべき所がないかも考えながらチェックしていく(大局的)

- 共有する. 戦略を錬る
- (一人)用意しておいたミドルウェアの設定ファイルを適用していく
- ガッとやる

- 計測する. 外の情報にもアンテナを貼っておく(OK)
