# TurtleBot3 Japan Custom
<img src="https://github.com/ROBOTIS-GIT/emanual/blob/master/assets/images/platform/turtlebot3/logo_turtlebot3.png" width="300">

## セットアップ手順（Quick Start Guide）
### 1. 環境設定
Turtlebot3の新FriendsモデルのPizzaを利用するには、まずTurtlebot3の基本的な設定を行います。そのため、Turtlebot3のE-manualの「[Quick Start Guide](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/)」を従って、セットアップを行ってください。ただし、以下の変更点に注意してください。

- [1.1 PC Setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#pc-setup)について

Pizzaモデルに関して、Raspberry PIの代わりにNUC11を使います。そして、現時点では、ROS Noeticに対応していますので、「[Quick Start Guide](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/)」の「Noetic」ブランチを選択してください。

- [1.1.4 Install TurtleBot3 Packages](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#install-turtlebot3-packages)の変更点

この部分では必要なROSパッケージをインストールしますが、今回はDebianのバイナルパッケージを使わずに、Robotis日本支店の公式GitHubからダウンロードします。以下の手順通りセットアップを進めてください。

```code
$ sudo apt remove ros-noetic-dynamixel-sdk
$ sudo apt remove ros-noetic-turtlebot3-msgs
$ sudo apt remove ros-noetic-turtlebot3
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/src/
$ git clone -b noetic-devel https://github.com/ROBOTIS-GIT/DynamixelSDK.git
$ git clone -b noetic-devel https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git
$ git clone -b noetic-jp-devel https://github.com/ROBOTIS-JAPAN-GIT/turtlebot3_jp_custom
$ cd ~/catkin_ws && catkin_make
$ echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

- [SBC Setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/sbc_setup/#sbc-setup)について

今回はRaspberry PIの代わりにNUC11を使いますので、E-manualに書いてあるステップを飛ばします。しかし、次のような手順でセットアップを行います。

1. NUCにUbuntu 20.04をインストールします。

2. ROS Noeticをインストールします。

3. Turtlebot3の必要なパッケージをインストールします。

```code
$ cd ~/catkin_ws/src/
$ git clone -b noetic-jp-devel https://github.com/ROBOTIS-JAPAN-GIT/turtlebot3_jp_custom
$ cd turtlebot3_jp_custom
$ rm -r turtlebot3_description/ turtlebot3_teleop/ turtlebot3_navigation/ turtlebot3_slam/ turtlebot3_example/
$ cd ~/catkin_ws && catkin_make
```

- [3.3 OpenCR Setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/opencr_setup)の変更点(更新！！！)

OpenCRを設定するには、NUCを通して、セットアップを行います。

1. [OpenCR](https://emanual.robotis.com/docs/en/parts/controller/opencr10/)をmicro USBケーブル経由でNUCに繋げてください。

1. OpenCRにファームウェアをアップロードするには、NUCに必要なDebianパッケージをインストールします。
```code
$ sudo dpkg --add-architecture armhf
$ sudo apt-get update
$ sudo apt-get install libc6:armhf
```

3. OPENCR_MODELに```pizza```と書きます。
```code
$ export OPENCR_PORT=/dev/ttyACM0
$ export OPENCR_MODEL=pizza_noetic
$ rm -rf ./opencr_update.tar.bz2
```

4. ファームウェアをダウンロードして、解凍します。
```code
$ wget https://github.com/ROBOTIS-JAPAN-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2 
$ tar -xvf opencr_update.tar.bz2 
```

5. OpenCRにファームウェアをアップロードします。
```code
$ cd ./opencr_update
$ ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr
```

6. Turtlebot3 Pizzaのファームウェアが正しくアップロードされたら、以下のようなメッセージが出力されます。
![OpenCR Success Output](https://myoctocat.com/assets/images/base-octocat.svg)


### ネットワーク設定
今までのTurtleBot3のモデルの中で、分散処理のためRaspberry PIが使われてきました。しかし、今回はその代わりにNUCを`master pc`として、そして別のPCをリモートパソコンとして用意することになりますので、以下のように設定してください。

- NUC（master PC）側
1. NUC PCが繋がれたWIFIのIPアドレスを取得します。
```code
$ ifconfig
```
「wlp2s0」の「inet addr」に書いてある`192.168.X.XXX`のようなIPをメモしておきます。
> **Note**
`192.168.X.XXX`の「X」という文字を各PCのIPに応じて変更してください。

2. 「~/.bashrc」のファイルにROS IPを設定します。

```code ~/.bashrc
$ nano ~/.bashrc
```
ドキュメントの一番下に以下のように書きます。
```
export ROS_MASTER_URI=http://192.168.X.XXX:11311
export ROS_HOSTNAME=192.168.X.XXX
```

3. 以上2行を追加したら、ファイルを保存するため、`Ctrl+s`（保存）の次に`Ctrl+x`（戻る）を押します。

4. 最後にROS IPの変更点をターミナルに反映させます。
```code
$ source ~/.bashrc
```

- リモートPC（remote PC）側

1. 前のNUC PCのIPアドレスを確認し、メモしておきます。（192.168.X.XXX）
1. リモートPC側のIPアドレスを`ifconfig`のコマンドで確認して、メモしておきます。（192.168.X.YYY）
1. 「~/.bashrc」のファイルにROS IPを設定します。

```code ~/.bashrc
$ nano ~/.bashrc
```
ドキュメントの一番下に以下のように書きます。
```
export ROS_MASTER_URI=http://192.168.X.XXX:11311
export ROS_HOSTNAME=192.168.X.YYY
```

4. 以上2行を追加したら、ファイルを保存するため、`Ctrl+s`（保存）の次に`Ctrl+x`（戻る）を押します。

5. 最後にROS IPの変更点をターミナルに反映させます。
```code
$ source ~/.bashrc
```

### 追加レポジトリの設定
Turtlebot3のPizzaモデルには、デフォルトとして「Realsense D435」のデプスカメラと「Sick Tim571」のLiDARが付けてあります。それそれのレポジトリのセットアップ手順を説明します。
> **Warning**
> 以下の手順はリモートPC側に行われます。

1. カメラ設定（要確認！！！）

まず、必要なパッケージをインストールします。
- ROS Wrapper for Intel® RealSense™ Devices（公式サイトより）
```code
$ sudo apt-get install ros-$ROS_DISTRO-realsense2-camera
$ cd ~/catkin_ws/src
$ git clone https://github.com/IntelRealSense/realsense-ros
$ cd ~/catkin_ws
$ catkin_make
```
> **Note**
> [realsense-ros](https://github.com/IntelRealSense/realsense-ros)の設定手順のついて詳しく公式のGitHubに参照してください。

次に、シミュレーション環境でもRealsense D435を使えるように、次のレポジトリもダウンロードします。
```code
$ cd ~/catkin_ws/src
$ git clone https://github.com/pal-robotics/realsense_gazebo_plugin
$ cd ~/catkin_ws
$ catkin_make
```


1. LiDAR設定

Sick Timのudev rulesの追加とともに、レポジトリをコンパイルします。

```code
$ cd ~/catkin_ws/src
$ git clone https://github.com/ROBOTIS-JAPAN-GIT/sick_tim
$ cd sick_tim
$ sudo cp debian/udev /etc/udev/rules.d/81-sick-tim3xx.rules
$ sudo udevadm control --reload-rules
$ cd ~/catkin_ws
$ cakin_make
```

> **Note**
> [sick_tim](https://github.com/uos/sick_tim)の公式GitHubとの差分はTim571のシミュレーション環境にも対応しています。

そして、LANケーブルでNUCに繋げるので、IPの設定も行います。

まず、Ubuntuの方の設定について、次の手順になります。
1. LiDARのIPを確認し、メモしててください
> **Note**
> デフォルトでIPは「192.168.0.1」になっていますが、別のIPアドレスを設定する必要な場合は、SICK公式サイトにて、IPの更新方法を確認してください。
2. Ubuntuで「Settings／設定」のプログラムを開きます。
1. 「Network／ネットワーク」に移動して、「Wired／有線」の一覧を確認します。
1. その最初の箱にあるマスミのイコンを押します。
1. IPv4に移動して、「IPv4 Method」はデフォルトで「Automatic (DHCP)」になっているところ、「Manual」を選択します。
1. 「Address」の中に、「Address」を「192.168.X.XXX」に、「Netmask」を「255.255.255.0」に、Gateway「192.168.X.1」にします。
> **Note**
> LiDARのIPに応じて、「X」の部分を更新します。そして、「XXX」の部分は、LiDARのIPと一致してはなりません。例えば、LiDARのIPは「192.168.0.1」の場合、「1」以外に「0~255」の間に番後を選択してください。

次に、「sick_tim571_2050101.launch」(~/catkin_ws/src/sick?tim)のlaunchファイルをつぎのように書き換えます。

- 更新前

```code
    <!-- Uncomment this to enable TCP instead of USB connection; 'hostname' is the host name or IP address of the laser scanner
    In cases where a race condition exists and the computer boots up before the TIM is ready, increase 'timelimit.'
         <param name="hostname" type="string" value="192.168.0.1" />
         <param name="port" type="string" value="2112" />
         <param name="timelimit" type="int" value="5" />
    -->
```

- 更新後
```code
    <!-- Uncomment this to enable TCP instead of USB connection; 'hostname' is the host name or IP address of the laser scanner
    In cases where a race condition exists and the computer boots up before the TIM is ready, increase 'timelimit.' -->
         <param name="hostname" type="string" value="192.168.X.XXX" />
         <param name="port" type="string" value="2112" />
         <param name="timelimit" type="int" value="5" />
```
> **Note**
> `value="192.168.X.XXX`の「X」の文字をLiDARのIPに応じて、更新します。例えば、デフォルトでLiDARのIPは「192.168.0.1」の場合、`value="192.168.0.1"`のままで大丈夫です。（別のIPアドレスを設定する必要な場合は、SICK公式サイトにて、IPの更新方法を確認してください。）

### 動かしてみよう！
この時点に付いたら、環境のセットアップは完了となります。これから、Turtlebot3 Pizzaモデルを実際に動かしてみましょう。動作方法は、E-manualの「[Bring-Up](https://emanual.robotis.com/docs/en/platform/turtlebot3/bringup/)」のままで進めば大丈夫です。

> **Note**
> sshよりNUCに繋ぐとき、NUCのIPアドレス（192.168.X.XXX）を使用することになります。

そして、新しいターミナルを開くたびに、Turtlebot3のモデルも指定してください。
```code 
$ export TURTLEBOT3_MODEL=pizza
```





## ハードウェア関係
### 部品リスト（BOM）
| 部品名 | 型番 | 個数 | 購入リンク |
|---|---|---|---|
| Dynamixel xm540-w150-r | 902-0134-000 | 2 | [here](https://e-shop.robotis.co.jp/product.php?id=42) |
| OpenCR1.0 | 903-0257-000 | 1 | [here](https://e-shop.robotis.co.jp/product.php?id=155) |
| NUC 11 Pro Kit NUC11TNHv7 | BNUC11TNHV70000 | 1 | [here](https://www.ark-pc.co.jp/i/31400996/) |
| TiM571-2050101 | 1075091 | 1 | [here](https://www.sick.com/jp/ja/detection-and-ranging-solutions/2d-lidar/tim/tim571-2050101/p/p412444) |
| Realsense d435 | --- | 1 | [here](https://www.intelrealsense.com/depth-camera-d435/) |
| 車輪(5inch) | --- | 2 | [here]() |
| なめらかオムニホイール（Φ55mm） | 4571398310089 | 2 | [here]() |
| アルミフレーム | CAF5-2020-400 | 4 | [here]() |
| アルミフレーム | CAF5-2020-360 | 6 | [here]() |
| アルミフレーム | CAF5-2020-170 | 4 | [here]() |
| アルミフレーム | CAF5-2020-100 | 5 | [here]() |
| 回り止付ハードブラケットSS | SFK-N58T | 52 | [here]() |
| Li-ionバッテリ 14.4V 9.0Ah 129.6Wh| BL1490 | 1 | [here](https://www.amazon.co.jp/dp/B08MHWMZ7C) |
| バッテリー18 vドック | B08X73Z7RP | 1 | [here](https://www.amazon.co.jp/dp/B08X73Z7RP) |
| スラスト針状ころ軸受 | BA0821 | 4 | [here]() |
| シェル形ニードルベアリング | TLA810Z | 2 | [here]() |
| 金属ワッシャ | TWSSS16-4-1  | 4 | [here]() |
| 黄銅スペーサー | BRB-435CE | 2 | [here]() |
| 六角穴付きボルト | CSH-SUS-M5-10 | 6 | [here]() |
| 六角穴付きボルト | CSH-SUS-M4-16 | 2 | [here]() |
| 六角穴付きボルト | SBCB3-8 | 2 | [here]() |
| 六角穴付きボルト | CSH-SUS-M2.5-20 | 8 | [here]() |
| 六角穴付きボルト | CSH-SUS-M2.5-12 | 8 | [here]() |
| 六角穴付ボルト UNC | CSH-SUS-UNC1/4-7/16 | 8 | [here]() |
| 六角穴付き皿ボルト | CSHCS-BR-M4-10 | 21 | [here]() |
| 六角穴付き皿ボルト | CSHCS-BR-M4-8 | 5 | [here]() |
| 六角穴付き皿ボルト | CSHCS-ST-M2.5-8 | 16 | [here]() |
| 六角穴付き皿ボルト | SHFBS3-10 | 2 | [here]() |
| ナット | LBNR4 | 4 | [here]() |
| ナット | LBNR2.5 | 8 | [here]() |
| 黄銅 ローレットツマミ | NB-310EA-N | 4 | [here]() |
| SF20 TナットSS（先入れ） | SFB-012  | 26 | [here]() |
| --- | --- | --- | [here]() |
| --- | --- | --- | [here]() |
| --- | --- | --- | [here]() |




### 3Dモデルの参照 (update link!!!)
詳細なパーツのリストとモデルの設計は、[Turtlebot3 Friends: Pizza](https://cad.onshape.com/documents/9d6a0395dffef67a3c72e937/w/9be45b8110f1eab640d1cdfd/e/9a65db04c65f7adb980422c1)のOnShapeドキュメントで確認できます。

### モデルの特徴
| 項目 | Pizza | Big Wheel |
|---|---|---|
| 最大直進速度 | 0.35 m/s | 0.50 m/s |
| 最大回転速度 | 1.49 rad/s | 3.41 rad/s |
| 最大ペイロード |  | 30kg |
| サイズ（長さ x 幅 x 高さ） | 434.94mm x 489.10mm x 261.54mm | 281mm x 306mm x 170.30mm |
| 重量 |  |  |
| 乗り上げ可能高さ |  |  |
| 動作時間 |  |  |
| 充電時間 |  |  |
| コンピュータ | NUC10i7FNHN | Raspberry Pi |
| MCP |  Intel® Core™ i7-10710U Processor (12M Cache, up to 4.70 GHz)  | 32-bit ARM Cortex®-M7 with FPU (216 MHz, 462 DMIPS) |
| リモートコントローラ | - | - |
| アクチュエータ | XM540-W150 | XM430-W210 |
| LiDAR | SICK TiM571 | 360 Laser Distance Sensor LDS-01 or LDS-02 |
| カメラ | Realsense D435 | Realsense D435 |
| IMU | Gyroscope 3 Axis
Accelerometer 3 Axis | Gyroscope 3 Axis
Accelerometer 3 Axis |
| 供給入力端子 |  | 3.3V / 800mA, 5V / 4A, 12V / 1A |
| I/Oピン |  | GPIO 18 pins, Arduino 32 pin |
| 周辺機器 |  | UART x3, CAN x1, SPI x1, I2C x1, ADC x5, 5pin OLLO x4 |
| DINAMIXELポート | RS485 x 3, TTL x 3 | RS485 x 3, TTL x 3 |
| オーディオ | 複数ブザービート使用可能 | 複数ブザービート使用可能 |
| LED使用可能 | User LED x 4 | User LED x 4 |
| LEDステータス | Board status LED x 1, Arduino LED x 1, Power LED x 1 | Board status LED x 1, Arduino LED x 1, Power LED x 1 |
| ボタンとスイッチ | Push buttons x 2, Reset button x 1, Dip switch x 2 | Push buttons x 2, Reset button x 1, Dip switch x 2 |
| バッテリ | Makita BL1040B 10.8V 4.0Ah | Lithium polymer 11.1V 1800mAh / 19.98Wh 5C |
| PC接続 | USB | USB |
| ファームウェア更新 | USB経由, JTAG経由 | USB経由, JTAG経由 |
| 電力アダプタ | Input : 100-240V, AC 50/60Hz, 1.5A @max, Output : 12V DC, 5A | Input : 100-240V, AC 50/60Hz, 1.5A @max, Output : 12V DC, 5A |

## ソフトウェア関係
### launchファイルの実行
### 選定デバイスの変更方法


[![kinetic-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/kinetic-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/kinetic-devel)

[![melodic-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/melodic-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/melodic-devel)

[![noetic-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/noetic-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/noetic-devel)

[![dashing-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/dashing-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/dashing-devel)

[![foxy-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/foxy-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/foxy-devel)

[![galactic-devel Status](https://github.com/ROBOTIS-GIT/turtlebot3/workflows/galactic-devel/badge.svg)](https://github.com/ROBOTIS-GIT/turtlebot3/tree/galactic-devel)

## ROBOTIS e-Manual for TurtleBot3
- [ROBOTIS e-Manual for TurtleBot3](http://turtlebot3.robotis.com/)

## Wiki for turtlebot3 Packages
- http://wiki.ros.org/turtlebot3 (metapackage)
- http://wiki.ros.org/turtlebot3_bringup
- http://wiki.ros.org/turtlebot3_description
- http://wiki.ros.org/turtlebot3_example
- http://wiki.ros.org/turtlebot3_navigation
- http://wiki.ros.org/turtlebot3_slam
- http://wiki.ros.org/turtlebot3_teleop

## Open Source related to TurtleBot3
- [turtlebot3](https://github.com/ROBOTIS-GIT/turtlebot3)
- [turtlebot3_msgs](https://github.com/ROBOTIS-GIT/turtlebot3_msgs)
- [turtlebot3_simulations](https://github.com/ROBOTIS-GIT/turtlebot3_simulations)
- [turtlebot3_applications_msgs](https://github.com/ROBOTIS-GIT/turtlebot3_applications_msgs)
- [turtlebot3_applications](https://github.com/ROBOTIS-GIT/turtlebot3_applications)
- [turtlebot3_autorace](https://github.com/ROBOTIS-GIT/turtlebot3_autorace)
- [turtlebot3_deliver](https://github.com/ROBOTIS-GIT/turtlebot3_deliver)
- [hls_lfcd_lds_driver](https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver)
- [turtlebot3_manipulation](https://github.com/ROBOTIS-GIT/turtlebot3_manipulation.git)
- [turtlebot3_manipulation_simulations](https://github.com/ROBOTIS-GIT/turtlebot3_manipulation_simulations.git)
- [open_manipulator_msgs](https://github.com/ROBOTIS-GIT/open_manipulator_msgs)
- [open_manipulator](https://github.com/ROBOTIS-GIT/open_manipulator)
- [open_manipulator_simulations](https://github.com/ROBOTIS-GIT/open_manipulator_simulations)
- [open_manipulator_perceptions](https://github.com/ROBOTIS-GIT/open_manipulator_perceptions)
- [dynamixel_sdk](https://github.com/ROBOTIS-GIT/DynamixelSDK)
- [OpenCR-Hardware](https://github.com/ROBOTIS-GIT/OpenCR-Hardware)
- [OpenCR](https://github.com/ROBOTIS-GIT/OpenCR)

## Documents and Videos related to TurtleBot3
- [ROBOTIS e-Manual for TurtleBot3](http://turtlebot3.robotis.com/)
- [ROBOTIS e-Manual for OpenManipulator](http://emanual.robotis.com/docs/en/platform/openmanipulator/)
- [ROBOTIS e-Manual for Dynamixel SDK](http://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/)
- [Website for TurtleBot Series](http://www.turtlebot.com/)
- [e-Book for TurtleBot3](https://community.robotsource.org/t/download-the-ros-robot-programming-book-for-free/51/)
- [Videos for TurtleBot3 ](https://www.youtube.com/playlist?list=PLRG6WP3c31_XI3wlvHlx2Mp8BYqgqDURU)
