# Graduation thesis
卒論「深層学習を用いたソフトウェアのバグ混入予測に関する研究」で使用したプログラム

# preprocess
学習に必要となるデータを収集し、前処理を実行するプログラムを置いている。

## code

### リポジトリの対応表の作成
OpenStackやQtは機能ごとにリポジトリが分けられていたため、先行研究が提供する再現データに記載されたコミットIDがどのリポジトリに存在するかを調べる必要があった。

1. repo_name.ipynbでコミットIDとリポジトリの対応表を作成

2. label_repo.pyで対応表を埋めていく
```
python label_repo.py repo_path label_csv
# repo_path: Qt（Openstack）プロジェクトの各リポジトリ
# label_csv: 1で作成した対応表
```

### 各コミットの情報を抽出
本研究では、各コミットにおいて、
1. 変更ファイルごとのソースコードの差分（追加行、削除行）
2. コミットメッセージ

## notebook

# learning