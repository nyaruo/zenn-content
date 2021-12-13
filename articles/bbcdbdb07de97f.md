---
title: "Flutterで学ぶSQLite(SQFlite)"
emoji: "🐬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "sqflite", "sqlite"]
published: false
---

# はじめに

https://www.youtube.com/watch?v=8bV9ixYNAL4&t=46s

https://thegrowingdeveloper.org/coding-blog/using-sqflite-in-flutter-application

https://crieit.net/posts/Flutter-SQLite#Flutter%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AESQLite

# sqfliteとは

sqfliteとは、Flutter用のSQLiteプラグインのことであり、Android/iOS/macOSに対応している。
デバイスにデータを保存する最も一般的な方法(ローカルDB)であり、SQLを用いて操作することができる。

https://pub.dev/packages/sqflite

# sqfliteの導入
Flutterプロジェクトに依存関係を追加する。
pubspec.yaml の dependencies: にsqfliteとpath_providerを記述する。
path_providerとは、Android,iOSとプラットフォームに依存することなく同じようなファイルへのアクセスを可能とするプラグインのことである。
https://pub.dev/packages/path_provider

```yaml: pubspec.yaml
dependencies:
  sqflite:
  path_provider:
```

```dart :main.dart
import 'package:flutter/material.dart';
import 'database_helper.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SQFLite Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {

  final dbHelper = DatabaseHelper.instance;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('SQFLite Demo Home Page'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            OutlinedButton(onPressed: () async{
              int i = await DatabaseHelper.instance.insert({
                DatabaseHelper.columnName : 'Saheb',
              });
              print('the inserted id is $i');
            }, child: Text('挿入(insert)')),
            OutlinedButton(onPressed: () async{
              List<Map<String, dynamic>> queryRows = await DatabaseHelper.instance.queryAll();
              print(queryRows);
            }, child: Text('読み取り(query)')),
            OutlinedButton(onPressed: () async {
              int updatedId = await DatabaseHelper.instance.update({
                DatabaseHelper.columnId: 12,
                DatabaseHelper.columnName: 'Mark',
              },);
              print(updatedId);
            }, child: Text('更新(update)')),
            OutlinedButton(onPressed: () async {
              int rowsEffected = await DatabaseHelper.instance.delete(13);
              print(rowsEffected);
            }, child: Text('削除(delete)')),
          ],
        ),
      ),
    );
  }
}
```

puspec.yamlに追加したプラグインを取得

```powershell
$ flutter pub get
```

取得できているか確認
```powershell
$ flutter pub deps
```

#

```dart :database_helper.dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';
// DatabaseHelperクラスを作成
// シングルトンクラスにすることで、データリークなどを防ぐことができる(トランザクション)
class DatabaseHelper{
  // DBの名前を定義
  static final _dbName = 'myDatabase.db';
  // DBのバージョンを指す(1または2がメジャーらしい)
  static final _dbVersion = 1;
  // テーブル名を定義
  static final table = 'myTable';
  // 列,カラム,属性(IDとnameとageの3つの列を用意)
  static final columnId = '_id';
  static final columnName = 'name';
  static final columnAge = 'age';

  // シングルトンクラス(インスタンスが単一)を作成する
  DatabaseHelper._privateConstructor();
  // static final: クラスで固有の定数を作成, instance: インスタンスを作成
  static final DatabaseHelper instance = DatabaseHelper._privateConstructor();

  // データベースを初期化(デバイスに初めてインストールされた際にDBにはnullが入る)
  static Database? _database;
  // null: https://i53.hatenablog.jp/entry/2021/01/07/011031
  // https://programming-dojo.com/%E3%80%90dart%E5%85%A5%E9%96%80%E3%80%91future%E3%82%92%E4%BD%BF%E3%81%86%E6%96%B9%E6%B3%95%E3%82%92%E8%AA%AC%E6%98%8E%E3%81%97%E3%81%BE%E3%81%99/
  // DBに接続する
  Future<Database?> get database async{
    // DBがnullでない場合は、DBのみを返す
    if(_database!=null) return _database;
    // そうでないなら、新しくDBし、新規DBを返す
    _database = await _initiateDatabase();
    return _database;
  }

  // DBを作成する関数
  _initiateDatabase() async{
    // アプリケーションのファイルが保存されているフォルダへのpathを取得(dart.ioが必要)
    Directory directory = await getApplicationDocumentsDirectory();
    // join: 取得したディレクトリpathとDBの名前を結合してpathに代入
    String path = join(directory.path, _dbName);
    // 指定したpathのDBを開く, なければ_onCreateを呼び出す
    return await openDatabase(path, version: _dbVersion, onCreate: _onCreate);
  }
  // _onCreate: DBが作成された際にテーブルを作成する
  Future _onCreate(Database db, int version) async{
    // CREATE TABLE: tableの作成, INTEGER PRIMARY KEY: 整数型の主キー, TEXT NOT NULL: テキスト型でNULLを除外する
    await db.execute('''
      CREATE TABLE $table(
      $columnId INTEGER PRIMARY KEY,
      $columnName TEXT NOT NULL,
      $columnAge INTEGER NOT NULL,
      )
      '''
    );
  }

  // 挿入
  // 文字列型と動的型のMap(行,レコード)を挿入する Map: 　https://qiita.com/hainet/items/daab47dc991285b1f552 , dynamic: https://note.com/hatchoutschool/n/n767701b099b0
  Future<int> insert(Map<String, dynamic> row) async{
    // ゲッター(Future<Database?> get database async)が呼び出される(DBに接続を行う)
    Database? db = await instance.database;
    return await db!.insert(table, row);
  }
  // 全件取得
  Future<List<Map<String, dynamic>>> queryAll() async{
    Database? db = await instance.database;
    return await db!.query(table);
  }

  // 特定の行数を取得
  Future<int?> queryRowCount() async{
    Database? db = await instance.database;
    return Sqflite.firstIntValue(await db!.rawQuery('SELECT COUNT(*) FROM $table'));
  }
  // 更新
  Future<int> update(Map<String, dynamic> row) async{
    Database? db = await instance.database;
    int id = row[columnId];
    // 特定のテーブルの任意のデータを更新, ?に指定した[id](= row[columnId]: 任意の行)が入る
    return await db!.update(table, row, where: '$columnId = ?', whereArgs: [id]);
  }

  // 削除
  Future<int> delete(int id) async{
    Database? db = await instance.database;
    return await db!.delete(table, where: '$columnId = ?', whereArgs: [id]);
  }
}
```


