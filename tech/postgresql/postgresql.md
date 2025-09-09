
# pg_basebackup.exe

```sh
d:\middleware\PostgreSQL\13\bin\pg_basebackup.exe -U postgres --pgdata=D:\CCSWORK\20250618\pg_basebackup
```

上記で取得したフォルダを別サーバーにコピーしてpostgresqlを起動する方法
1. リストア先のpostgresqlを停止する
2. リストア先にバックアップをフォルダごとコピーする
3. コピーしたフォルダの権限に``Network Service``を追加しフルコンを付与する

## 統計情報のリセット
pg_basebackup.exeで取得したフォルダを別サーバーに移動した
統計情報が取得できなかった(pg_stat_all_tables等)

pg_stat_databaseにstat_resetという列がある
統計情報をリセットされた日付が格納される

selectしてみると復元した日付になってる、、、orzl

###### 再起動
postgresql.conf中のstats_temp_directoryに設定されたディレクトリに
統計情報を書き出しておいて、通常の再起動では統計情報がリセットされないようになっている

デフォルト値: pg_stat_tmp

## reindex
REINDEXは、インデックスの中身を1から作り直すという点では、インデックスを削除してから再作成する処理と似ています。 しかし、ロックに関しては異なります。 REINDEXはインデックスの元となるテーブルの書き込みをロックしますが、読み込みはロックしません。 また、処理中のインデックスに対する排他ロックを取得するので、そのインデックスを使用する読み込みはブロックされます。 一方、DROP INDEXは瞬間的に元となるテーブルの排他ロックを取得するので、書き込みも読み込みもブロックされます。 その後に行うCREATE INDEXでは書き込みのみをロックし、読み込みはロックしません。 インデックスは存在しないので、インデックスを使用する読み込みは発生しません。 したがって、読み込みがブロックされることはありませんが、コストが高いシーケンシャルスキャンの使用を強制されることになります。
```
REINDEX { INDEX | TABLE | DATABASE | SYSTEM } _name_ [ FORCE ]
```
## 物理ファイル
クラスタ内の各データベースに対して、PGDATA/base 内にサブディレクトリが存在し、サブディレクトリ名は pg_database 内のデータベース OID となります。 このサブディレクトリはデータベースファイルのデフォルトの位置であり、特にシステムカタログがそこに格納されます。

各テーブルおよびインデックスは別個のファイルに格納され、ファイル名はテーブルまたはインデックスの_ファイルノード_番号となり、pg_class.relfilenode 内で検索できます。

| **注意**                                                                                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| テーブルにおけるファイルノード番号と OID は多くの場合一致しますが、常に一致するとは_限らない_ことに注意してください。 TRUNCATE、REINDEX、CLUSTER 等の幾つかの操作、および ALTER TABLE における幾つかの形式は、OID を保持したままファイルノード番号を変更できます。 ファイルノード番号とテーブル OID が同一であると仮定しないでください。 |

テーブルまたはインデックスが 1 ギガバイトを超えると、ギガバイト単位の_セグメント_に分割されます。 最初のセグメントのファイル名はファイルノード番号と同一であり、それ以降は、filenode.1、filenode.2、等の名称となります。 この配置法によってファイル容量に制限のあるプラットフォームにおける問題を回避します。

## 拡張モジュール
PostgreSQLの拡張モジュールとは、==データベースの機能を拡張するためのモジュール==です。PostgreSQL本体とは別に開発され、追加の関数、演算子、データ型などを提供します。

#### 確認
``` sql
SELECT * FROM pg_extension;
or
\dx
```
#### 作成
``` sql
CREATE EXTENSION [ IF NOT EXISTS ] extension_name;
```
作成可能な拡張は
```sql
select * from pg_available_extensions;
```

## トランザクションID(XID)
32bitの数値。現在の値と比較して行の可視・不可視を決定する。
一周すると全ての行が見えなくなる。
#### 確認
```sql
-- 実行すると1進む
select txid_current();
```
テーブル内で凍結されていないXID
``` sql
-- 3列目は現在のXIDとの差
select relname, relfrozenxid, age(relfrozenxid) from pg_class;
```

#### 通常のID
2以上の値
#### 特殊なID
- 0: Invalid Transaction ID
- 1: BootstrapTransactionId
- 2: FrozenTransactionId

#### XID = 2に関して
バージョン9.4から動作が変更された様だ
現在はFrozenTransactionIdはフラグを立てるだけと説明されている
https://www.postgresql.jp/docs/9.6/routine-vacuuming.html

#### 比較
modulo-2^32
っていう数式らしい、詳細は分からん

#### FREEZE(凍結)
どのトランザクションIDからも古いと認識されるようにする事
実態はフラグを立てる事のようだ

## 論理構造
- データベースクラスタ
	- データベース
		- スキーマ
			- テーブル等のオブジェクト 
# 新しいデータベースクラスタ
```cmd
E:\middleware\PostgreSQL\13\bin>"E:\middleware\PostgreSQL\13\bin\initdb.exe" -D "S:\CCSWORK\ueno\data"
データベースシステム内のファイルの所有者はユーザ"CATS13DEV"となります。
このユーザをサーバプロセスの所有者とする必要があります。

データベースクラスタはロケール"Japanese_Japan.932"で初期化されます。
ロケールにより暗黙的に指定される符号化方式"SJIS"はサーバ側の
符号化方式として使用できません。
デフォルトのデータベース符号化方式は代わりに"UTF8"に設定されます。
initdb: ロケール"Japanese_Japan.932"用の適切なテキスト検索設定が見つかりませんでした
デフォルトのテキスト検索構成は simple に設定されます。

データベージのチェックサムは無効です。

ディレクトリS:/CCSWORK/ueno/dataを作成しています ... ok
サブディレクトリを作成しています ... ok
動的共有メモリの実装を選択しています ... windows
デフォルトのmax_connectionsを選択しています ... 100
デフォルトのshared_buffersを選択しています ... 128MB
デフォルトの時間帯を選択しています ... Asia/Tokyo
設定ファイルを作成しています ... ok
ブートストラップスクリプトを実行しています ... ok
ブートストラップ後の初期化を実行しています ... ok
データをディスクに同期しています ... ok

initdb: 警告: ローカル接続に対して"trust"認証を有効にします
pg_hba.confを編集する、もしくは、次回initdbを実行する時に -A オプション、
あるいは --auth-local および --auth-host オプションを使用することで変更する
ことがきます。

成功しました。以下のようにしてデータベースサーバを起動することができます:

    ^"E^:^\middleware^\PostgreSQL^\13^\bin^\pg^_ctl^" -D ^"S^:^\CCSWORK^\ueno^\data^" -l ログファイル start
```

## 新しいクラスタをwindowsサービスに登録
```
d:\middleware\PostgreSQL\13\bin\pg_ctl.exe register -N mypostgres -D E:\CCSWORK\ueno\data.org
```
