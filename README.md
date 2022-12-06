[![ROS-noetic-build-test](https://github.com/CIT-Autonomous-Robot-Lab/livox_camera_lidar_calibration/actions/workflows/build.yaml/badge.svg)](https://github.com/CIT-Autonomous-Robot-Lab/livox_camera_lidar_calibration/actions/workflows/build.yaml)

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
git clone https://github.com/CIT-Autonomous-Robot-Lab/livox_camera_lidar_calibration.git
cd ..
catkin build
source devel/setup.bash
```

## 2. カメラの内部パラメータのキャリブレーション
### 2-1 準備
以下のキャリブレーションボードを印刷し、厚紙などに貼り付けてください。  
その後、キャリブレーションボードの写真を撮影してください。少なくとも20枚の写真が必要です。
<div align="center">
<img src="https://github.com/YuwaAoki/livox_camera_lidar_calibration/blob/master/doc_resources/chess_board.png">
</div>

<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/calibphotoc.png">
</div>

### 2-2 カメラキャリブレーション  
MATLABを使用する方法か、このパッケージを用いた方法のいずれかを行ってください。
#### 2-2-1 MATLABキャリブレーション
MATLABのツール「cameraCalibrator」を使用してキャリブレーションできます。「RadialDistortion」「TangentialDistortion」「K」のパラメータが必要です。
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/matcalib.png">
</div>

#### 2-2-2 ROSキャリブレーション
パッケージ内のノードを使用してキャリブレーションできます。
txtファイルに使用する写真の名前を保存してください。写真名を書き込む際には、以下のシェルスクリプトが有効です。  
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/intxt.png">
</div>

```
#! /bin/bash

rm in.txt && touch in.txt

for ((i=0; i<=31; i++))
do
        echo -e "$i.png" >> in.txt
done
```
camera_in_pathで写真の名前が書き込まれたファイル、camera_folder_pathで写真が保存された場所、result_pathで結果を保存するファイル、row_numberでキャリブレーションボードの行数、col_numberで列数、widthとheightでグリッドの幅と高さを指定してください。  
「row_number」と「col_number」は実際の数値から1引いたものを設定するとうまく動作する気がします。  
```
roslaunch camera_lidar_calibration cameraCalib.launch camera_in_path:="$(rospack find camera_lidar_calibration)/../../data/camera/in.txt" camera_folder_path:="$(rospack find camera_lidar_calibration)/../../data/camera/photos/" result_path:="$(rospack find camera_lidar_calibration)/../../data/camera/result.txt" row_number:="9" col_number:="6" width:="20" height:="20"
```

### 2-3 パラメータの保存
得られたパラメータは次の形式でtxtファイルに書き込んでください。distortionの数値のうち、左の２つがRadialDistortionで、右の３つがTangentialDistortionです。

<div align="center">
<img src="https://github.com/YuwaAoki/livox_camera_lidar_calibration/blob/master/doc_resources/intrinsic_format.png">
</div>

## 3. カメラとLiDARデータの準備
### 3-1 キャリブレーションシーンの準備
キャリブレーションを行うシーンを準備します。ターゲットポイントを選択するため開けた環境と、写真と点群で目印になるものを用意してください。このステップでもLiDARとカメラを目印から3m離してください。以下の写真は千葉工業大学津田沼キャンパス2号館3階で撮影されたものです。　　
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/calibscene.png">
</div>   

### 3-2 LiDARの調整  
キャリブレーションボードの点群が表示されているかをコマンドで確認します。  
```
roslaunch livox_ros_driver livox_lidar_rviz.launch
```
rosbag記録時はこちらのコマンドを使用します。
```
roslaunch livox_ros_driver livox_lidar_msg.launch
```
[次のリンク](https://github.com/Livox-SDK/livox_ros_driver)を参照して、「customMsg」形式のデータが含まれていることを確認してください。確認できたら、カメラを準備してください。  
### 3-3 カメラとLiDARデータの収集
1. 写真を取る  
2. 点群を記録
```
roslaunch livox_ros_driver livox_lidar_msg.launch
rosbag record /livox/lidar
```
3. データごとに写真とrosbagを10秒記録します。

## 4. 校正データ取得
### 4-1 写真の座標の取得
起動ファイルを実行します。intrinsic_pathでカメラのパラメータを保存したファイル、input_photo_folder_pathで使用する写真の場所、output_pathで結果を保存するファイルを指定してください。
```
roslaunch camera_lidar_calibration cornerPhoto.launch intrinsic_path:="$(rospack find camera_lidar_calibration)/../../data/parameters/intrinsic.txt" input_photo_folder_path:="$(rospack find camera_lidar_calibration)/../../data/photo" output_path:="$(rospack find camera_lidar_calibration)/../../data/corner_photo.txt"
```
写真が表示されたら、写真上でターゲットポイントを選択します。通常キャリブレーションボードの左上隅から反時計回りに選択します。ポイントを4つ選択したら、最後に適当なポイントを選択して完了です。座標を記録し直す場合は、保存した座標を消去してからやり直してください。  
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/corner_photo.png">
</div>   

### 4-2 点群の座標の取得
rosbagをpcdファイルに変換します。input_bag_pathでrosbagが保存された場所、output_pcd_pathでpcdファイルを保存する場所、data_numでrosbagファイルの数を指定してください。
```
roslaunch camera_lidar_calibration pcdTransfer.launch input_bag_path:="$(rospack find camera_lidar_calibration)/../../data/lidar/" output_pcd_path:="$(rospack find camera_lidar_calibration)/../../data/pcdFiles/" data_num:="1" 
```
pcdファイルを開き、写真と同じ順序でポイントを選択します。shift+左クリックでポイントを選択できます。
```
pcl_viewer -use_point_picking xx.pcd
```
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/corner_lidar.png">
</div>   

取得した座標はtxtファイルに書き込んでください。  

<div align="center">
<img src="https://github.com/YuwaAoki/livox_camera_lidar_calibration/blob/master/doc_resources/corner_lidar.png">
</div>  

## 5. 外部パラメータ取得
以下のコマンドを実行して、外部パラメータを計算します。intrinsic_pathでカメラのパラメータを保存したファイル、extirnsic_pathで結果を保存するファイル、input_lidar_pathで点群の座標を記録したファイル、input_photo_pathでカメラの座標を記録したファイルを指定してください。
```
roslaunch camera_lidar_calibration getExt1.launch intrinsic_path:="$(rospack find camera_lidar_calibration)/../../data/parameters/intrinsic.txt extrinsic_path:="$(rospack find camera_lidar_calibration)/../../data/parameters/extrinsic.txt input_lidar_path:="$(rospack find camera_lidar_calibration)/../../data/corner_lidar.txt" input_photo_path:="$(rospack find camera_lidar_calibration)/../../data/corner_photo.txt" error_threshold:="12"
```
誤差が大きいデータは端末に出力されます。

## 6. 結果の検証
コマンドを実行してrvizで確認します。intrinsic_pathでカメラのパラメータを保存したファイル、extirnsic_pathで外部パラメータを保存したファイル、input_bag_pathでrosbagファイル、input_photo_pathで写真を指定してください。
```
roslaunch camera_lidar_calibration colorLidar.launch intrinsic_path:="$(rospack find camera_lidar_calibration)/../../data/parameters/intrinsic.txt" extrinsic_path:="$(rospack find camera_lidar_calibration)/../../data/parameters/extrinsic.txt" input_bag_path:="$(rospack find camera_lidar_calibration)/../../data/lidar/0.bag" input_photo_path:="$(rospack find camera_lidar_calibration)/../../data/photo/0.png"
```
<div align="center">
<img src="https://github.com/CIT-Autonomous-Robot-Lab/livox_cam_lidar_calib_document-ja/blob/master/doc_resouces/colorlidar.png">
</div>
