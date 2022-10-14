# Livox_cam_lidar_calib_document-ja
LivoxのカメラとLiDARのROSキャリブレーションパッケージの日本語ドキュメントです。  
画像と点群データから非リアルタイムなキャリブレーションを行います。  
なお、パッケージではLivox製LiDARのみをサポートしています。

## 1 環境構築
ubuntu20.04での動作を確認済み。パッケージはubuntu16.04で開発されています。

### 1-1 ドライバーのインストール
ubuntu20.04の場合は、Livox-SDKの代わりにLivox-SDK2をインストールしてください。
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
