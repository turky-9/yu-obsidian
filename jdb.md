# jdb
javaのコマンドラインデバッガ

## path
%JAVA_HOME%/bin/jdb.exe

## help
``` sh
# help
jdb -help

# list connector(共有メモリやTCPポート等、接続方法が色々ある)
jdb -listconnectors
```

## Tomcatにアタッチ
mdb.exeと異なりプロセスにアタッチしていてもリクエストを中断することなく処理し続ける

``` sh
# 最後の7560はプロセス番号なので適宜書き換えてください
jdb -connect com.sun.jdi.ProcessAttach:pid=7560
```

#### socketでのアタッチ
下記を起動するとGUIが表示される
"D:\middleware\a$ache-tomcat-9.0.13\bin\tomcat9w.exe"

以下の設定値が確認できる
```
-agentlib:jdwp=transport=dt_socket,address=localhost:8000,server=y,suspend=n
```
ポート8000でデバッガをアタッチ出来るような設定となっている

``` bat
jdb -connect com.sun.jdi.SocketAttach:port=8000
```
### アタッチ後
アタッチするとプロンプトが表示される
``help``でヘルプが表示される
抜粋
```
catch [uncaught|caught|all] <class id>|<class pattern>
                          -- 指定された例外が発生したときにブレークします
ignore [uncaught|caught|all] <class id>|<class pattern>
                          -- 指定された例外の'catch'を取り消します
use (or sourcepath) [source file path]
                          -- ソース・パスを表示または変更します
```


``` sh
# とりあえずブレークしそうなところにブレークポイントを張りまくる
catch uncaught *Exception
stop at groupcats.common.workflow.jigo.servlet.remote.UpdateExecUserID:86
stop at groupcats.common.workflow.plugin.servlet.remote.GetUserID:435
stop at groupcats.common.workflow.jigo.servlet.remote.UpdateExecUserID:101
stop at groupcats.common.workflow.plugin.servlet.remote.GetUserID:555
stop at groupcats.common.workflow.plugin.servlet.remote.GetUserID:444
stop at groupcats.common.workflow.plugin.servlet.remote.GetUserID:386
stop at groupcats.common.workflow.plugin.servlet.remote.GetUserID:151
```

すると最終承認で以下がブレークした
``` sh
444             String s_cstacttypecd = (String) g_actinf.getValue("cstacttypecd");
ajp-nio-127.0.0.1-8009-exec-5[1]

# 色々コマンドを実行してみる
ajp-nio-127.0.0.1-8009-exec-5[1] locals
メソッド引数:
x_opercd = 0
ローカル変数:
s_fainal = "FAINAL_APPROVAL"
s_approver = "APPROVER"
s_chkErrMsg = "「起票部門承認」と「最終承認」は同一者で承認出来ません。"
s_netErrMsg = "HTTPレスポンス取得が失敗しました！"

ajp-nio-127.0.0.1-8009-exec-5[1] where
  [1] groupcats.common.workflow.plugin.servlet.remote.GetUserID.checkExecAction (GetUserID.java:444)
  [2] jdk.internal.reflect.NativeMethodAccessorImpl.invoke0 (nativeメソッド)
  [3] jdk.internal.reflect.NativeMethodAccessorImpl.invoke (NativeMethodAccessorImpl.java:62)
  [4] jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke (DelegatingMethodAccessorImpl.java:43)
  [5] java.lang.reflect.Method.invoke (Method.java:566)
  [6] jp.co.sei.RakWF21.doc.sub.rkadPluginLoader.invokePlugin (rkadPluginLoader.java:426)
  [7] jp.co.sei.RakWF21.doc.sub.rkadPluginLoader.invokePlugin (rkadPluginLoader.java:389)
  [8] jp.co.sei.RakWF21.plugin.rkawExecPlugin.invokePluginCore (rkawExecPlugin.java:2,251)
  [9] jp.co.sei.RakWF21.plugin.rkawExecPlugin.invokeCheckPlugin (rkawExecPlugin.java:2,168)
  [10] jp.co.sei.RakWF21.plugin.rkawExecPlugin.checkExecAction (rkawExecPlugin.java:1,681)
  [11] jp.co.sei.RakWF21.ptn.screen.rkaw3012.check (rkaw3012.java:1,285)
  [12] jp.co.sei.is.lib21.pms.ptn.PtnObject.checkError (PtnObject.java:5,441)
  [13] jp.co.sei.is.lib21.pms.ptn.PtnObject.exec (PtnObject.java:3,963)
  ・・・以下略


ajp-nio-127.0.0.1-8009-exec-5[1] list
440
441             String s_netErrMsg = "HTTPレスポンス取得が失敗しました！";
442
443             // 個別アクティビティタイプCDを取得
444 =>          String s_cstacttypecd = (String) g_actinf.getValue("cstacttypecd");
445             // アクティビティタイプ名を取得する場合
446             // String s_acttypenm = (String) g_actinf.getValue("acttypenm");
447
448             // 一般承認と経理審査以外の場合は、承認者チェック
449             // if ((s_cstacttypecd.indexOf(s_accounting) < 0) && !(s_general.equals(s_cstacttypecd))) {

# printコマンドは優秀
ajp-nio-127.0.0.1-8009-exec-5[1] list
447
448             // 一般承認と経理審査以外の場合は、承認者チェック
449             // if ((s_cstacttypecd.indexOf(s_accounting) < 0) && !(s_general.equals(s_cstacttypecd))) {
450             // 最終承認のみチェック
451 =>          if (s_cstacttypecd.indexOf(s_fainal) >= 0) {
452                     // 査閲・承認のみ、同一承認者チェック
453                     if (x_opercd == 0) {
454                             // ログインユーザーIDを取得
455                             String p_loginUser = (String) g_ssp.getUserID();
456
ajp-nio-127.0.0.1-8009-exec-5[1] print s_cstacttypecd.indexOf(s_fainal)
 s_cstacttypecd.indexOf(s_fainal) = 0


# 順番にステップ実行していくとヌルポがcatchされている事が分かった
catch all java.lang.NullPointerException


ajp-nio-127.0.0.1-8009-exec-8[1] locals
メソッド引数:
jsonArray = instance of org.json.JSONArray(id=10836)
code = "KANRINO"
name = "伝票管理番号"
ローカル変数:
jsonObjItem = instance of org.json.JSONObject(id=10839)
ajp-nio-127.0.0.1-8009-exec-8[1] print g_docfv
 g_docfv = "jp.co.sei.RakWF21.plugin.rkawDocFormValue@2f2a6539"
ajp-nio-127.0.0.1-8009-exec-8[1] print g_docfv.getValue(code)
 g_docfv.getValue(code) = null
```

フォームの値が取得できる変数「g_docfv」から``KANRI_NO``の値が取得できないようだ

# 別件
以下、おかしくないか！？
会社コードが定数として埋まっている、、、
GetUserID.java
``` java
	public int checkStartWorkflow(SeiAryValue x_route) {
		// 一般社員承認のアクティビティタイプコード
		String s_general = "GENERAL_APPROVAL";
		// 起票部門承認のアクティビティタイプコード
		String s_internal = "INTERNAL_APPROVAL";
		// 箇所承認(JX金属本社)のアクティビティタイプコード
		String s_jx03001 = "JX030-01_APPROVAL";
```
