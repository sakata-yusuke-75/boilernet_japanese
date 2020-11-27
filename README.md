# boilernetの日本語対応
boilernetを日本語対応したモデルです。
googletrendsデータセットをgoogle翻訳で日本語にしたものとそれを用いて学習したモデルを公開しています。
訓練時のパラメータやデータセットの詳細などは元のgithubをご参照ください。
https://github.com/mrjleo/boilernet

## Requirements
This code is tested with Python 3.7.5 and
* tensorflow==2.1.0
* numpy==1.17.3
* tqdm==4.39.0
* nltk==3.4.5
* beautifulsoup4==4.8.1
* html5lib==1.0.1
* scikit-learn==0.21.3
* transformer==3.1.0
* scipy==1.4.1


## Usage
以下のデータセットについて使用可能です。
* GoogleTrends-2017

また以降のコードを動かす前にnltkのダウンロードを済ませておかないとパースに失敗します。
```
python -c "import nltk; nltk.download('punkt')
```

前処理と学習の方法については元論文を参照してください。
ここでは追加した日本語文書学習方用の引数と日本語文書の予測方法について記述します。

### Preprocessing
```
usage: preprocess.py [-h] [-s SPLIT_DIR] [-w NUM_WORDS] [-t NUM_TAGS]
                     [--save SAVE]
                     DIRS [DIRS ...]
                     [-td TRAIN_DIR][-l LANGUAGE]

positional arguments:
  DIRS                  A list of directories containing the HTML files

optional arguments:
  -h, --help            show this help message and exit
  -s SPLIT_DIR, --split_dir SPLIT_DIR
                        Directory that contains train-/dev-/testset split
  -w NUM_WORDS, --num_words NUM_WORDS
                        Only use the top-k words
  -t NUM_TAGS, --num_tags NUM_TAGS
                        Only use the top-l HTML tags
  --save SAVE           Where to save the results
  -td TRAIN_DIR, --train_dir TRAIN_DIR
                        訓練時に前処理済みファイルを保存したディレクトリです。この引数が存在する場合はこのディレクトリの語彙を用いてDIRSのデータを前処理します。
  -language LANGUAGE
                        この引数にJapaneseが入っているとトークナイザーを日本語対応のものに切り替えます。デフォルトはEnglishです。
```
学習の際はtrain_dirの引数は入れないでください。
予測の際はtrain_dirに訓練に用いたにディレクトリを,DATA_DIRに予測したいhtmlファイルが入ったディレクトリを指定してください。
以下に一例を載せます。
```
python3 net/preprocess.py ~/source/ -w 5000 -t 50 --save ~/local_data_japan -td ~/googletrends_japanese_data_5000 -j Japanese
```

### Training
学習のスクリプトの使用方法は元コードから変更が無いため割愛します。

### Prediction
```
usage: preprocess.py [-w WORKING_DIR] [-i PREDICT_INPUT_DIR]
                     [-o PREDICT_OUTPUT_DIR] 
                     [-r PREDICT_RAW_DIR]
                     [-p CHECKPOINT_NUMBER]
                     [-b BALANCE]

arguments:
  -w WORKING_DIR
          訓練時に用いたWORKING_DIR
  -i PREDICT_INPUT_DIR
          前処理済みの予測したいデータのディレクトリ
  -o PREDICT_OUTPUT_DIR
          予測後の本文のテキスト情報、削除されたテキスト情報を出力するディレクトリ
  -r PREDICT_RAW_DIR
          前処理前の予測したいデータのディレクトリ
  -p CHECKPOINT_NUMBER
          予測に用いるモデルの保存されているチェックポイントの番号
  -b BALANCE
          予測値に足し合わせる数
          0.4にすると、本来0.5以上で本文と予測される所を0.1以上で本文と予測されるようになります
```
使用例としては以下のようになります。
```
python net net/predict.py -w ~/googletrends_japanese_train_5000 -i ~/local_data_japan -o ~/日本語対応_recall調整版 -r ~/source/ -p 9
```

