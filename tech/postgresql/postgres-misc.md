# misc
###### 現在のDB

``` sql
select current_database();
```
###### DB list
``` sql
\l
```
###### change db
``` sql
\c dbname
```
###### スキーマ一覧
```sql
\dn
```
###### テーブルの詳細
```sql
\dS+ table_name
```
###### show
現在のパラメータ(postgresql.conf)を表示する
```sql
show param_name
```

# スロークエリ
``` sql
-- 確認
select * from pg_available_extensions where name = 'pg_stat_statements';

-- installed_versionが空なら要インストール
CREATE EXTENSION pg_stat_statements;


SELECT * FROM pg_stat_statements;
```
pg_stat_statementsモジュールは、サーバで実行されたすべてのSQL文の実行時の統計情報を記録する手段を提供します。

このモジュールは追加の共有メモリを必要とするため、postgresql.confのshared_preload_librariesに
pg_stat_statementsを追加してモジュールをロードしなければなりません。
このことは、このモジュールを追加もしくは削除するには、サーバを **再起動** する必要があるということを意味しています。
[[postgresql#拡張モジュール]]

postgresql.conf
```
shared_preload_libraries = 'pg_stat_statements'
```

``` sql
SELECT query,calls,total_exec_time,min_exec_time,max_exec_time,mean_exec_time
FROM pg_stat_statements order by mean_exec_time desc;

--reset
SELECT pg_stat_statements_reset();
```

# 現在の接続の状態
``` sql
select * from pg_stat_activity
```
state列が現在の状態

# ロック(lock)
``` sql
select * from pg_locks

select pg_blocking_pids(1880)
```
pg_locks.granted == trueなら獲得済
falseならロック待ち

# キュッシュヒット
https://lets.postgresql.jp/documents/technical/statistics/2

## データベース全体
キャッシュヒット率の確認
pg_stat_databaseのblks_readとblks_hitを利用します。
blks_hitは共有メモリにあった(キャッシュヒットした)ブロックの読み込み回数、
blks_readは共有メモリに無かった(キャッシュヒットしなかった)ブロックの読み込み回数です。
ここでいうブロックとは、PostgreSQLの最小のI/O単位である8kB(デフォルトの場合)の塊を指します。
ほぼ同義で"ページ"とも言われます。
blks_hit*100/(blks_hit+blks_read) の計算でキャッシュヒット率を算出します。
メモリ等のチューニングでは、これをなるべく100に近づけていくことを目指します。
``` sql
-- division by zero エラー (0割り) に注意しましょう
SELECT datname,  
round(blks_hit*100/(blks_hit+blks_read), 2) AS cache_hit_ratio  
FROM pg_stat_database WHERE blks_read > 0;
```

## テーブル毎
pg_statio_all_tablesだとシステムテーブルも含まれる
``` sql
SELECT relname,
   round(heap_blks_hit*100/(heap_blks_hit+heap_blks_read), 2)
   AS cache_hit_ratio FROM pg_statio_user_tables
     WHERE heap_blks_read > 0 ORDER BY cache_hit_ratio desc;
```

## インデックス毎
```sql
SELECT relname, indexrelname,
   round(idx_blks_hit*100/(idx_blks_hit+idx_blks_read), 2)
   AS cache_hit_ratio FROM pg_statio_user_indexes
     WHERE idx_blks_read > 0 ORDER BY cache_hit_ratio desc;
```
# ガベージ
ガベージ※の量の確認方法
pg_stat_user_tablesのn_live_tup(有効な行数)とn_dead_tup(ガベージとなった行数)を利用します。
下記の様にガベージの行数を割合とテーブルのサイズを併せて出力すると、
ガベージがより具体的な量として把握できます。
``` sql
SELECT relname, n_live_tup, n_dead_tup,
round(n_dead_tup*100/(n_dead_tup+n_live_tup), 2)  AS dead_ratio,
pg_size_pretty(pg_relation_size(relid)) FROM pg_stat_user_tables
WHERE n_live_tup > 0 ORDER BY dead_ratio DESC;
```

※PostgreSQLは、更新処理や削除処理を行った場合、古いデータは消さずに残しておきます。
この古いデータ = ガベージ

# ガベージ2
```sql
select schemaname,
	relname,
	n_dead_tup,
	n_live_tup,
	round(n_dead_tup::float/n_live_tup::float*100)dead_pct,
	autovacuum_count,
	last_vacuum,
	last_autovacuum,
	last_autoanalyze,
	last_analyze
from pg_stat_all_tables
where n_live_tup >0;
```
# テーブル毎のサイズ
```sql
SELECT pgn.nspname,
    relname,
    pg_size_pretty(relpages::bigint * 8 * 1024) AS size,
    CASE WHEN relkind = 't' THEN (
            SELECT pgd.relname
            FROM pg_class pgd
            WHERE pgd.reltoastrelid = pg.oid)
        WHEN nspname = 'pg_toast' AND relkind = 'i' THEN (
            SELECT pgt.relname
            FROM pg_class pgt
            WHERE SUBSTRING(pgt.relname FROM 10) = REPLACE(SUBSTRING(pg.relname FROM 10), '_index', ''))
        ELSE (
            SELECT pgc.relname
            FROM pg_class pgc
            WHERE pg.reltoastrelid = pgc.oid)
    END::varchar AS refrelname,
    CASE WHEN nspname = 'pg_toast' AND relkind = 'i' THEN (
        SELECT pgts.relname
        FROM pg_class pgts
        WHERE pgts.reltoastrelid = (
            SELECT pgt.oid
            FROM pg_class pgt
            WHERE SUBSTRING(pgt.relname FROM 10) = REPLACE(SUBSTRING(pg.relname FROM 10), '_index', '')
            ))
    END AS relidxrefrelname,
    relfilenode,
    relkind,
    reltuples::bigint,
    relpages 
FROM pg_class pg, pg_namespace pgn
WHERE pg.relnamespace = pgn.oid
-- AND pgn.nspname NOT IN ('information_schema', 'pg_catalog')
ORDER BY relpages DESC; 
```

