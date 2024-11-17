# Graduation thesis
卒論「深層学習を用いたソフトウェアのバグ混入予測に関する研究」で使用したプログラム

## 環境
Python: 3.9.19

ライブラリ: requirements.txt

## preprocess
学習に必要となるデータを収集し、前処理を実行するプログラムとデータを置いている

### リポジトリの対応表の作成
OpenStackやQtは機能ごとにリポジトリが分けられていたため、先行研究が提供する再現データに記載されたコミットIDがどのリポジトリに存在するかを調べる必要があった。

1. `repo_name.ipynb`でコミットIDとリポジトリの対応表を作成

2. `label_repo.py`で対応表を埋めていく
```
python label_repo.py (1)　(2)
# (1): Qt（Openstack）プロジェクトの各リポジトリ
# (2): 1で作成した対応表(openstack_reponame.csv, qt_reponame.csv)
```
出力データ：preprocess/resource/preprocess_data/openstack_reponame.csv, qt_reponame.csv

### 各コミットの情報を抽出
本研究では、各コミットにおいて、
1. 変更ファイルごとのソースコードの差分（追加行、削除行）(#code)
2. コミットメッセージ [cm]
3. コミットメッセージに含まれるITSデータ（Issue Tracking Sysytemのデータ）[its]

のデータを収集した。
以下にその手順を示す。
1. `get_commit_diff.py`によって、各コミットの情報（コード差分、コミットメッセージ）を取得していく
```
python get_commit_diff.py -csf_filename (1) -project (2) -json_name (3) -auth_ext (4)
# (1):リポジトリとコミットIDの対応表ファイル (openstack_reponame.csv, qt_reponame.csv)
# (2):プロジェクトの名前　例：openstack, qt
# (3):結果の出力ファイル名
# (4):ソースファイルとみなす拡張子
```
出力データ：resource/preprocess_data/oepnstack_repo_inf.json　または　qt_repo_inf.json

2. `extract_issue_id.py`でコミットメッセージからIssueIDを抽出
```
python extract_issue_id.py (1) (2)
# (1):preprocess/resource/openstack_repo_inf.json または qt_repo_inf.json
# (2):出力ファイル名
```
出力データ：resource/preprocess_data/openstack_its.json または qt_its.json

3. `scrape_from_its.py`でIssueIDの情報を取集
```
python scrape_from_its.py　(1) (2) (3)
# (1)：resource/preprocess_data/openstack_its.json または qt_its.json
# (2)：出力ファイル名
# (3)：プロジェクト名（openstack または qt）
```
出力データ：resource/preprocess_data/openstack_its_inf.json, qt_openstack_its_inf.json

### データセットの作成
RQとしてITSデータを含めたコミットメッセージとそうでない場合のデータにおける予測性能の比較を行う。そのため、データセットは各プロジェクトにおいて2種類作成する。

プログラムは、`make_dataset.py`であり、
ITSデータをコミットメッセージに含める場合（1）(2) (3) (4) (5)を標準入力に渡し、そうでない場合 (1) (2) (3) を標準入力に渡す。
```
python make_dataset.py (1) (2) (3) (4) (5)
# (1)：resource/preprocess_data/openstack_repo_inf.json または qt_repo_inf.json
# (2)：resource/deepjit_data/openstack.csv または qt.csv
# (3)：プロジェクト名（openstack または qt）
# (4)：resource/preprocess_data/openstack_its_inf.json または qt_its_inf.json
# (5)：resource/preprocess_data/openstack_its.json または qt_its.json
```
出力データ：

OpenStack

ITSデータを含めた場合:

resource/in_its_data/openstack_codes_dict.json, openstack_msg_dict.json, openstack_train.pkl, openstack_test.pkl

ITSデータを含めない場合:

resource/no_its_data/openstack_codes_dict.json, openstack_msg_dict.json, openstack_train.pkl, openstack_test.pkl

Qt

ITSデータを含めた場合:

resource/in_its_data/qt_codes_dict.json, qt_msg_dict.json, qt_train.pkl, qt_test.pkl

ITSデータを含めない場合:

resource/no_its_data/qt_codes_dict.json, qt_msg_dict.json, qt_train.pkl, qt_test.pkl


## learning
学習用のプログラムとデータを置いている
卒論のRQではソースコードの差分モデル、コミットメッセージのモデルにわけ、先行研究との比較結果を検証した。

Research Questionに関係がないため、論文には記載していないがメトリクスデータのモデルを加えた3つのモデルで最終的にアンサンブルも行っていた。

### 学習プログラム
`run.py`が学習のメインプログラムである。

```
run.py -i_tr_data (1) -i_te_data (2) -s (3) -c (4) -p (5) -i_m_dict (6) -i_c_dict (7)
#(1): 訓練データセット（openstack_train.pkl　または　qt_train.pkl）
#(2): テストデータセット（openstack_test.pkl　または　qt_test.pkl）
#(3): 学習モデルの指定 （lgb, code_cnn, msg_tf, ensemble）
#(4): 交差検証方法の設定（random：ランダム, time：時系列）
#(5): プロジェクトの指定 （openstack　または qt）
#(6): コミットメッセージのコーパスデータセット（openstack_msg_dict.json または qt_msg_dict.json）
#(7): ソースコード片のコーパスデータセット(openstack_code_dict.json または qt_msg_dict.json)
```

データ：

in_its： ITSデータあり

no_its： ITSデータなし

学習時のログ情報：log/general.log, general2.log

テスト時のログ情報：log/result.log, result2.log

学習時のベストモデル：in_its/best_model、no_its/best_model

予測結果：in_its/pred, no_its/pred

分析結果：notebook/
