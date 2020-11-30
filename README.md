# boilernetの日本語対応
boilernetを日本語対応したモデルです。
googletrendsデータセットをgoogle翻訳で日本語にしたものとそれを用いて学習したモデルを公開しています。
訓練時のパラメータやデータセットの詳細などは[BoilerNet](https://github.com/mrjleo/boilernet)のgithubをご参照ください。
また本コードはPython 3.7.5 で動作します。必要なライブラリの詳細はrequirements.txtをご参照ください。

## Usage
本文抽出ツールとしての使用方法について記述します。

また以降のコードを動かす前にnltkのダウンロードを済ませておかないとパースに失敗します。(エラーがtry文の中で止まるので気付きにくいです。)
```
python -c "import nltk; nltk.download('punkt')
```

前処理と学習の詳細については元論文を参照してください。
ここでは追加した日本語文書学習用の引数と日本語文書の予測方法について記述します。
大きく以下の3つのステップです。

1. 予測したいhtmlファイルを格納したディレクトリを用意
2. 上記をpreprocess.pyで前処理
3. 前処理結果をpredict.pyで予測

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
                        訓練時に前処理済みファイルを保存したディレクトリです。この引数が存在する場合はこのディレクトリの辞書を用いてDIRSのデータを前処理します。
  -language LANGUAGE
                        この引数にJapaneseが入っているとトークナイザーを日本語対応のものに切り替えます。デフォルトはEnglishです。
```
学習の際はtrain_dirの引数は入れないでください。
予測の際はtrain_dirに訓練に用いた前処理済みデータセットが保存されているディレクトリを,DATA_DIRに予測したいhtmlファイルが入ったディレクトリを指定してください。
以下に一例を載せます。
```
python3 net/preprocess.py ~/predict_source/ -w 5000 -t 50 --save ./preprocessed/predict_data -td ./preprocessed/googletrends_japanese_data -j Japanese
```

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
          WORKING_DIR内のtrain.csvを見て精度の良いモデルを選択してください
  -b BALANCE
          予測値に足し合わせる数
          0.4にすると、本来0.5以上で本文と予測される所を0.1以上で本文と予測されるようになります
```
使用例としては以下のようになります。
```
python3 net/predict.py -w ./working/googletrends_japanese_train -i ./preprocessed/predict_data -o ~/predict_output -r ~/predict_source/ -p 9
```

-oで指定した出力ディレクトリに抽出された本文(filename_predict.txt)と除外された文章(filename_delete.txt)のtxtファイルが別々に出力されます。

また再学習を行う際の学習方法は以下に記述します。ファイルの保存場所などは少し変更していますが大筋の変更点はほぼ無いため、詳細はBoilerNetのgithubをご覧ください。

### Training
```
usage: train.py [-h] [-l NUM_LAYERS] [-u HIDDEN_UNITS] [-d DROPOUT]
                [-s DENSE_SIZE] [-e EPOCHS] [-b BATCH_SIZE]
                [--interval INTERVAL] [--working_dir WORKING_DIR]
                DATA_DIR

positional arguments:
  DATA_DIR              Directory of files produced by the preprocessing
                        script

optional arguments:
  -h, --help            show this help message and exit
  -l NUM_LAYERS, --num_layers NUM_LAYERS
                        The number of RNN layers
  -u HIDDEN_UNITS, --hidden_units HIDDEN_UNITS
                        The number of hidden LSTM units
  -d DROPOUT, --dropout DROPOUT
                        The dropout percentage
  -s DENSE_SIZE, --dense_size DENSE_SIZE
                        Size of the dense layer
  -e EPOCHS, --epochs EPOCHS
                        The number of epochs
  -b BATCH_SIZE, --batch_size BATCH_SIZE
                        The batch size
  --interval INTERVAL   Calculate metrics and save the model after this many
                        epochs
  --working_dir WORKING_DIR
                        Where to save checkpoints and logs
```

使用例は以下です。
```
python3 net/train.py ./preprocessed/googletrends_japanese_data/ -e 50 --working_dir ./working/googletrends_japanese_train
```