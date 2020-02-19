# git_for_handover_materials
2020年卒　飛翔班，中村翔太の引き継ぎ用Git．

*** 

## 新規クアッドロータ（赤いF330）用メインGit：https://github.com/doraiden555/drone_git_kai.git
Gitのツリーからわかるように，UWB_opt_control-01.pyとUWB_control_withouopt-01.pyが最新コードである．


この2つを主に改良していくべし．


### UWB_opt_control-01.py
「UWB＋オプティカルフロー」を用いて位置推定を行った上で位置を制御するコードである．


2020年2月に行った修論審査会で述べたように位置制御ゲイン（PID）の調整が甘いことや，プロペラの推力が弱いことから，この制御は未完成なのでブラッシュアップすべし．75行目から78行目までがピッチ，ロール，高さ，ヨーコントロールのゲインを表す．


主にピッチとロールのゲインをいじれば位置制御の応答性が変わる．高さゲインに至っては失敗を恐れ，調整が未完なため，慎重に調整すべし．


次に，コードの大まかな説明をする．


  - 初めにもシュールのインポートやカルマンフィルタや各関数などに用いられる変数を初期化した後，43行目辺りにて4つのアンカの絶対座標を入力する．将来的にUWBで絶対座標を得るならここも関数化してどうこうしてください．


  - 80行目から88行目においてPIDコントロールようのクラスオブジェクトを生成する．クラスの定義は参考にした[DronePilot](https://github.com/doraiden555/DronePilot.git)のモジュールを参照して．


  - uart()関数にてTeensyLCからUart通信にて送られてきたデータをそれぞれ変数に格納する．


  - startup()関数ではDD_oldの初期化やスレッド化したuart()関数を動かすなどする．


  - IMU()にてジャイロセンサのキャリブレーションをする．


  - yaw_calibration()にてヨー角の初期化を行う．初めにクアッドロータを置いた向きを0度として扱う．


  - yaw_filter()では推定されたヨー角の補正を行う．


  - pos_cal()にて位置の初期化を行う．初めにクアッドロータを置いた地点を位置(x, y)=(0, 0)として扱う．


  - log()はCSVロギング用の関数．


  - pos_estimate()では初期化した初期位置を引数にクアッドロータの位置の推定を行う．カルマンフィルタを用いて推定しており，そこそこ複雑なので，詳細は私の修論や赤堀さんの修論を参考に．


  - control()関数では推定された位置を引数に位置の制御を行う．アルゴリズムは修論に書いたように推定された位置と目標位置の偏差よりPIDコントローラを組み，姿勢角を求める．先のDronePilotのpix-hover-controller.pyをほぼパクったので参考に．プロポのチャンネル"5"のPPWM幅が"1000"以上の場合，ピッチロールなどの各チャンネルをオーバーライドし，自動制御モードに入る．"1000"以下の場合はオーバーライドをクリアし，手動制御を受け付けるようになる．この場合は1点のみの位置制御であるが，もし3点を移動する位置制御がしたい場合は，456行目からのコメントを外すことで，それが可能である．チャンネル"7"を三段階に切り替えることで，位置の目標値が変わる仕組みである．プロポの設定はチャンネル"7"が三段スイッチに振られているためこれが可能である．


  - あとはメイン関数であるが，このコードを実行することでメイン中に記した関数が順に実行される仕組みである．


### UWB_control_withouopt-01.py
上のコードからカルマンフィルタにおける，オプティカルフローの観測項を抜いただけなので基本は同じ．
しかし，観測行列が異なるため，分散の値も調整しなければならない．

### GPS_test_UWB4.py
当時，赤堀さん（2018年3月卒）が書いた位置推定用のプログラム．内容を見ればわかるが，位置推定のアルゴリズムはほぼこれをパクっている．

***

## TeensyLC，Arduino用Git：https://github.com/doraiden555/arduino_git.git
### Tag6.ino
ArduinoのTag（クアッドロータに乗ったやつ）に書き込む用のプログラム.


計測されたUWBの距離補正とか発散値の処理とか色々やってるが自由に書き換えOK.


ただし，毎度のごとくアンカのIDは書き換えると面倒くさそう．


### uart_uwb_opt_2.ino
TeensyLC用のプログラム．シリアル通信をPC，Arduino，NAVIO2（Raspberry Pi）とそれぞれ行う．


本プログラム内でオプティカルフローセンサ（SPI）距離センサ（I2C）から送られてくるデータをもとに
移動距離（単位は距離）を計算する（詳しくは修論）．


シリアル通信で繋がったArduinoからは上のプログラムを用いた上で，各アンカとの距離データ（4つ）が送られてくるので，
移動距離の数値（オプティカルフローより）とで配列に格納した上，PC及びNAVIO2に送る．

***

## ステレオカメラプログラム用Git：https://github.com/doraiden555/stereo_camera_git.git
詳しい使用方法は本GitのREADME参照．

ただ，修論で述べたように，クアッドロータが飛行中における奥行方向の推定値はハチャメチャなので，真値の計測方法に至っては大幅に改良の余地がある．位置計測アルゴリズム（プログラムであったりマーカーの検出方法であったり）やチェスボードを用いたキャリブレーションの方法など．

この正確な真値測定方法だけで軽い論文一本書けるくらいの労力は必要であると思われる．それくらい正確な真値を計測するのは難しい．

青木が言っていたようにx-y平面内の位置だけなら天井に取り付けた広角カメラの1台で測定可能な気がする（方法は知らん）

***

## 修論用Git：https://github.com/doraiden555/Syuron_git.git
名前の通り2020年3月卒業，中村翔太の卒論．

***

## SII用Git：https://github.com/doraiden555/git_for_SII.git
2020年1月にハワイにて開催された[SII](https://sice-si.org/conf/SII2020/)という学会の資料．

***

## 飛翔ミーティング及びインフラミーティングの資料：https://github.com/doraiden555/tex_git.git
飛翔ミーティングにて作成した資料．どういう経緯で研究が進んだのか参考に．


