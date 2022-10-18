# livox_cam_lidar_calib_document-ja
LivoxのカメラとLiDARのキャリブレーションROSパッケージの日本語ドキュメントです。  
画像と点群データから非リアルタイムなキャリブレーションを行います。  
なお、パッケージではLivox製LiDARのみをサポートしています。  
[サンプルデータ](https://terra-1-g.djicdn.com/65c028cd298f4669a7f0e40e50ba1131/Download/update/data.zip)を使う場合、ws_livox内に展開してください。

## 1. 環境構築
ubuntu20.04での動作を確認済み。パッケージはubuntu16.04で開発されています。
### 1-1 ドライバーのインストール
ubuntu20.04の場合は、Livox-SDKの代わりにLivox-SDK2をインストールしてください。  
また、LiDARとの接続テストは各リポジトリを参照してください。
```
# Livox_SDKのインストール
git clone https://github.com/Livox-SDK/Livox-SDK.git
cd Livox-SDK
cd build && cmake ..
make
sudo make install
```
```
# Livox_SDK2のインストール
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake .. && make -j
sudo make install
```
```
# livox_ros_driverのインストール
mkdir ws_livox
cd ws_livox && mkdir src
git clone https://github.com/Livox-SDK/livox_ros_driver.git ./src
catkin_make
```  
### 1-2 依存関係のインストール
- [Point Cloud Library](https://pointclouds.org/downloads/)
- [Eigen](https://eigen.tuxfamily.org/dox/GettingStarted.html)
- [Ceres-solver](http://ceres-solver.org/installation.html)
### 1-3 ソースコードのダウンロードとコンパイル
```
# install camera/lidar calibration package
cd src
git clone https://github.com/Livox-SDK/livox_camera_lidar_calibration.git
cd ..
catkin build
source devel/setup.bash
```
open-cvのバージョンが4.0以降の場合、以下を使用してください。
```
cd src
git clone https://github.com/kaanoguzhan/livox_camera_lidar_calibration.git
cd ..
catkin build
source devel/setup.bash
```

## 2. カメラの内部パラメータのキャリブレーション
### 2-1 準備
以下のキャリブレーションボードを印刷し、厚紙などに貼り付けてください。  
また、カメラの視野とLiDARの視野を一致させてください。その後、キャリブレーションボードの写真を撮影してください。少なくとも20枚の写真が必要です。
<div align="center">
<img src="https://github.com/YuwaAoki/livox_camera_lidar_calibration/blob/master/doc_resources/chess_board.png">
</div>

### 2-2 カメラキャリブレーション  
#### 2-2-1 MATLABキャリブレーション
MATLABのツール「cameraCalibrator」を使用してキャリブレーションできます。「RadialDistortion」「TangentialDistortion」「K」のパラメータが必要です。
#### 2-2-2 ROSキャリブレーション
パッケージ内のノードを使用してキャリブレーションできます。
cameraCalib.launchのパラメータを設定してください。デフォルトではdata/camera/photo内の写真と、data/camera/in.txt内の写真名を参照します。「row_number」と「col_number」は実際の数値から1引いたものを設定するとうまく動作する気がします。
```
roslaunch camera_lidar_calibration cameraCalib.launch
```
