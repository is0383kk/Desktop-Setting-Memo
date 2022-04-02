デスクトップ環境構築
====
# PCスペック
iiyamaのDeep ∞　（ディープインフィニティ）  
導入OS:Ubuntu20.04   
CPU:core i7-8700  
GPU:Geforce GTX 1080 Ti 11GB  
SSD:240GB  
DDR4-2666 DIMM 16GB  

# Ubuntu20.04版  
## 手順0:Universal USB installerでISOファイルをUSBに移す  
[UbutuISOファイルダウンロード](https://www.ubuntulinux.jp/News/ubuntu2004-ja-remix)  
「Ubuntu」を選択，ダウンロードしたISOファイル「ubuntu-ja-20.04.1-desktop-amd64」を選択  
## 手順1:Ubuntu・NVIDIAドライバのインストール
Ubuntu-USBを指し，Ubuntuのダウンロードを行い再起動．  
`sudo apt-get update`及び`sudo apt-get upgrade`を実行．  
※「設定->電源」にてブランクスクリーンを「しない」に変更． 端末を開き「メニュー->名前なし」から「端末ベルを鳴らす」を切る．   
左下から「アプリケーションを表示する」にて「追加のドライバー」を選択する．  
NVIDIAの最新のドライバーを選択して「変更の適用」．「設定->このシステムについて」にて「グラフィック」でGPUが反映されていることを確認し`reboot`．（再起動するとログインループになると思うので下部の「その他」を参考に修復）  


## CUDAの導入  
[CUDA](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local)  
[今までのCUDA](https://developer.nvidia.com/cuda-toolkit-archive)  
「Linux->x86_64->Ubuntu->20.04->deb(local)」を選択し下部の指示に従いCUDAを導入．  
導入後，`nvidia-smi`で確認．  
確認後，`.bashrc`に以下を記述．  
```~/.bashrc
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```
`source .bashrc` コマンドでシェルの設定を反映させる。

[NVIDIAドライバ・CUDAの設定](https://qiita.com/SwitchBlade/items/5d5bc645822229ee0ed9)  


# その他，エラー処理や不具合対応など
## ログインループに陥った際の対処法（いい加減どうにかならんかな）  
[Ubuntuログインループ対処法](https://musaprg.hatenablog.com/entry/2020/06/30/201759)  
1. Ctrl+Alt+F2を押して仮想コンソールに入る  
2. `sudo vi /etc/default/grub`でgrubの設定を開く．※「x」で一文字消去．「:q!」でファイルを保存せず閉じる．「:wq」でファイルを保存し閉じる  
4. `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`から、`splash`を消す。 変更後は`GRUB_CMDLINE_LINUX_DEFAULT="quiet"`となっているのを確認  
5. `sudo update-grub`で変更を適用し，`reboot`













# 以下旧手順 
## 手順0:Universal USB installerでISOファイルをUSBに移す  
[UbutuISOファイルダウンロード](https://www.ubuntulinux.jp/News/ubuntu2004-ja-remix)  
「Ubuntu」を選択，ダウンロードしたISOファイル「ubuntu-ja-20.04.1-desktop-amd64」を選択  
## 手順1
0. BIOS設定  
起動時にF2連打でBIOS画面に移行  
"Advanced"で「SATA operation」を「AHCI」に「USB Wake Support」を「Enabled」に変更  
**「Security」でセキュアブートの無効化（Disabled）**  
これをしないとUbuntuが起動しなくなったりする．  
"Boot"で「File Browser Add Boot Option」で「USB」を選択．「EFI」，「BOOT」，「grubx64.efi」の順に選択し適当に名前を付ける．  
起動優先順位を先ほど名前を付けたものを一番上にし，F10で再起動．  
1. インストール時  
ubuntuインストール時に「Try Ubuntu without install」を選択する画面で**eを押し**grubを編集する．  
`quit splash`を`pci=nomsi quiet splash nomodeset `に変更する．  
2. grubの編集  
/etc/default/grub をエディタで開き  
    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`  
    `GRUB_CMDLINE_LINUX=""`  
上記を以下に変更する．  
    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=nomsi nomodeset"`  
    `GRUB_CMDLINE_LINUX="pci=noaer"`  
そして再起動する．  


## 手順2:GPUのドライバをダウンロード
以下のURLからドライバーをダウンロード  
[ドライバーダウンロード](//www.nvidia.com/Download/index.aspx)  
1. GeForce  
2. Geforce 10 Series  
3. Geforce GTX 1080 Ti  
4. Linux 64-bit  
5. Japanese  
## 手順3:Ubuntu自体のドライバを無効化(Ubuntu19.10ではこの作業はなくてもできた)
1. **/etc/modprobe.d/**内に**blacklist-nouveau.conf**というファイルを作成  
    `blacklist nouveau`  
    `options nouveau modeset=0`  
と書き込み保存  
`$ sudo update-initramfs -u`  
で再読み込みし**再起動**
## 手順4:インストーラの実行(Ubuntu19.10ではgdm3の設定を変えて対処)
1. **Ctrl + Alt + F1 でcuiモードに入る**  
`$ sudo service lightdm stop`  
でGUIを停止  
2. 実行権限を付与した後に実行．  
`$ sudo ./NVIDIA-3XXXXXX(任意の名前).run`  
`$ nvidia-smi`  
でGPUの認識を確認する  
3. `$ sudo service lightdm restart`  
でGUIモードに入り*再起動*  

# CUDAおよびcuDNNの設定
## 手順1:CUDAのダウンロード
[CUDA](https://developer.nvidia.com/cuda-90-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal)  
1. Linux  
2. x86_64  
3. Ubuntu  
4. 16.04  
5. deb(local)  
## 手順2:インストール
1. インストール  
`$ sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb`  
`$ sudo apt-key add /var/cuda-repo-9-0-local/7fa2af80.pub`  
`$ sudo apt-get update`  
`$ sudo apt-get install cuda`  
その後，パッチをあてる  
`$ sudo dpkg -i パッチ`  
2. .bashrcに書き込み  
    `export PATH=/usr/local/cuda-9.0/bin:${PATH}`  
    `export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64:${LD_LIBRARY_PATH}`  
再起動し**nvcc**，**nvidia-smi**などで確認する．  
## 手順3:cuDNNのダウンロード
[cuDNN](https://developer.nvidia.com/rdp/cudnn-download)  
1. CUDA9.0に対応するものをダウンロード．  
2. **Runtime Library for Ubuntu**と**Developer Library for Ubuntu**の両方をダウンロード  
`$ sudo dpkg -i `  
でインストール．  

# 追記(Ubuntu19.10インストール時)
Ubuntu19.10インストール時にも同様にログインループが発生  
## 解決手順:NVIDIA driverの導入
とりあえず，update，upgrade  
NVIDIA driverを探す．（必要ないらしいけどした）  
`$ sudo add-apt-repository ppa:graphics-drivers/ppa`  
`$ sudo apt update`  
`$ ubuntu-drivers devices`  
でNVIDIA driverを探す  
recommendedされているものを以下のコマンドでインストール  
`$ sudo apt install nvidia-driver-xxx`  
`$ nvidia-smi`  
で確認．  
**ただし，このまま reboot するとログインループになる**  
  そこで，以下を実行する(ドライバインストール直後に実行で解決した)  
1. gdm3を設定ごとアンインストール  
`$ sudo apt-get purge gdm3`  
2. gdm3を再インストールする  
`$ sudo apt-get install gdm3`  
その後 reboot  
### gdm3について
GDMはGNOME Display Managerの略  
ログインループの原因はこれにある．  
ログイン画面に影響を及ぼすファイルが以下のファイル．  
`$ /etc/gdm3custom.conf`  
以下の項目が自動ログインの項目とログインするIDの項目  
`AutomaticLoginEnable= Bool`  
`AutomaticLogin=UserID`  
**もし環境構築完了後に再起動した際にログインループに陥る場合もこの手順で解決可能**
