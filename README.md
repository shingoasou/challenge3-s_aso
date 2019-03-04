* Ruby 2.3.4p301 (2017-03-30 revision 58214) [x86_64-linux]
* Rails 4.2.5
* RubyGems 2.6.11
* sqlite 3.8.2

![2019-03-03 22 58 03](https://user-images.githubusercontent.com/13845927/53696555-6fa29880-3e0b-11e9-9633-55fd3524b47f.png)


## 課題に対するアプローチ
 ① 機能の整理
 * CSV取り込み、DB登録、集計項目の抽出と可視化
 * 爆速でCSV取り込み、DB登録、DBからのデータを使って可視化まで、Herokuにデプロイできるかどうか確認する
  （環境構築からデプロイまでの流れを検証。local環境では動作している）
 * gemなどの整理はしていない
 * security alertメールが飛ぶ可能性あり（bundle update activejob などした）

 ② テストケースの洗い出し  
 CSVファイル読み込み結果を検証するテストケース
 
  * テーブルのレコードをクリアする
  * ファイルが存在することをチェックする
  * 0バイトファイルでないことをチェックする
  * ファイルを読み込みテーブルに登録する
  * テーブル登録結果を検証する

 CSVファイルの読み込み(異常系)を検証するテストケース
  * 存在しないファイルを指定した場合はメッセージを出力して終了する
  * ログファイルをクリアする
  * テーブルのレコードをクリアする
  * ファイルパスを作成する
  * ファイルが存在しないことをチェックする
  * ファイルを読み込みテーブルに登録する
  * テーブル登録結果を検証する
  
  CSVファイルのカラムを検証するテストケース
  * カラムの変数名と大文字・小文字のチェック
  * 型、NULL、空文字の扱い（エラー処理 etc）

  データ可視化を検証するテストケース
  * 期待されるグラフが表示される
  * 値が合っている

 ③ Railsを扱うのが1年ぶりであること、cloud9の設定が古いことから、ひとまず以下のアプローチをとることにした。
  1. 参考サイトの整理

  * https://qiita.com/m-shin/items/7e8cdea2644e47b87fbf
  * https://ruby-rails.hatenadiary.com/entry/20141120/1416483136
  * https://qiita.com/succi0303/items/745fddfc3689a867759c
  * https://kyopa.hatenablog.com/entry/2018/10/10/140321
  * https://gorails.com/episodes/charts-with-chartkick-and-groupdate
  * https://qiita.com/kazukimatsumoto/items/a0daa7281a3948701c39  
  
  2. 1.が終了次第、challenge3のCSVデータを使って、①の手順で問題ないことを確認する  
  * カラムの数と型をチェック  
  * 2つのCSVを扱うためのMVC作成およびロジック実装  
  * グラフ描画ライブラリChartkick実装（このライブラリは色々とできて、面白い）  

　3. リファクタリングとテスト  
  * データが間違っていないか  
  * Model：データ（行、列）追加可能で使える状態にする  
  * View（見栄え）  
  * 保守性とパフォーマンス（グラフの種類の切り替えの速さ、グラフの数が多くなっても処理が遅くならないかなど）  

  * インデックス  
　　検索対象になるカラムにはインデックスを加える  
　　add_index :energies, :year unique: true, name: 'year_index'  

  * csv  
  * ファイルがないときの例外処理file.pathのあるなし  
  * nil, 空文字、改行などのバリデーション  
  * ファイルアップロードの時間を短くする  

　4. Herokuデプロイ  
  * Herokuにデプロイは成功したが、heroku run rake db:migrate をすると、  
  * /app/vendor/bundle/ruby/2.5.0/gems/activesupport-4.2.5/lib/active_support/core_ext/numeric/conversions.rb:131:in `block (2 levels) in <class:Numeric>’
  のエラーが出る。  
  * RubyおよびRailsのバージョンが古い？DBの設定もなんか臭う。環境構築からきちんとしたほうがよさそう。  

　5. PR提出

　6. リリース後の運用  
　* カラムの型変更などに対応  
　* 本番用のデータベースを試験環境にコピーして、必ず予行演習を行う  
　* DBのバックアップもしておく  
　* 失敗した場合の対処の手順（バックアップからDB復旧、アプリケーションのバージョンを戻す）もドキュメント化しておくこと  
　* テスト環境で実際に失敗の予行演習を行う  
　* 集計項目の追加  
　* 例えば、ボタンを押すと、その条件でグラフが描画される  
　* 一覧表示から特定の条件で検索した場合（例：2011年）の検索結果一覧とグラフ表示機能もおそらく追加していくことになるだろう  
　* グラフの整備  
　* 縦軸、横軸、ラベルなど  

---------------------------------------------------------------------------
 Next ToDo
  * AWS Cloud9に更新 or Docker
  * Gemの整理・更新
  * EnergyProductionの予測精度を高めるモデル構築（説明変数：Daylightなど）
  * HouseとEnergyのアソシエーション
   - City別Year別のEnergyProduction
   - MonthごとのEnergyProduction（City別, Year別）
  * 利益を最大化する施策
   - 電力売電単価のデータと利益状況の可視化
   - 蓄電・放電（いつ売電する？でも長い間蓄電できない）
   - コスト（イニシャルとランニング）
