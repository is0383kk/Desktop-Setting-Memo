# Ubuntu20.04 LTS導入＋GPUドライバ導入＋CUDA導入
機械学習用デスクトップ環境構築マニュアル（自分用）  
CUDAの導入まで  

PCスペック：  
iiyamaのDeep ∞　（ディープインフィニティ）  
導入OS:Ubuntu20.04   
CPU:core i7-8700  
GPU:Geforce GTX 1080 Ti 11GB  
SSD:240GB  
DDR4-2666 DIMM 16GB  

# Ubuntu20.04版(2022/04/01導入済み)  
## Universal USB installerでISOファイルをUSBに移す  
[UbutuISOファイルダウンロード](https://www.ubuntulinux.jp/News/ubuntu2004-ja-remix)  
Universal USB installer起動後「Ubuntu」を選択，ダウンロードしたISOファイル「ubuntu-ja-20.04.1-desktop-amd64」を選択  


## BIOS設定  
起動時にF2連打でBIOS画面に移行  
Alienwareの場合："Advanced"で「SATA operation」を「AHCI」に「USB Wake Support」を「Enabled」に変更(iiyamaPCはどうだったか忘れた)  
**「Security」でセキュアブートの無効化（Disabled）**  
これをしないとUbuntuが起動しなくなったりする．  
"Boot"で「File Browser Add Boot Option」で「USB」を選択．「EFI」->「BOOT」->「grubx64.efi」の順に選択し適当に名前を付ける．  
起動優先順位を先ほど名前を付けたものを一番上にし，F10で再起動．  


## Ubuntu・NVIDIAドライバのインストール
Ubuntu-USBを指し，Ubuntuのダウンロードを行い再起動．  
`sudo apt-get update`及び`sudo apt-get upgrade`を実行．  
※「設定->電源」にてブランクスクリーンを「しない」に変更． 端末を開き「メニュー->名前なし」から「端末ベルを鳴らす」を切る．   
左下から「アプリケーションを表示する」にて「追加のドライバー」を選択する．  
NVIDIAの最新のドライバーを選択して「変更の適用」．「設定->このシステムについて」にて「グラフィック」でGPUが反映されていることを確認し`reboot`．（再起動するとログインループになると思うので下部の「その他」を参考に修復）  


## CUDAの導入  
**注意：[PyTorch](https://pytorch.org/get-started/locally/)から適切なCUDAのVersionを調べた上でインストール**  
[CUDA11.6](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local)  
[今までのCUDA](https://developer.nvidia.com/cuda-toolkit-archive)  
「Linux->x86_64->Ubuntu->20.04->deb(local)」を選択し下部の指示に従いCUDAを導入．  
導入後，`nvidia-smi`で確認．  
確認後，`.bashrc`に以下を記述．  
```~/.bashrc
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```
`source .bashrc` コマンドでシェルの設定を反映させる．  
[NVIDIAドライバ・CUDAの設定参考元](https://qiita.com/SwitchBlade/items/5d5bc645822229ee0ed9)  


# その他，エラー処理や不具合対応など
## ログインループに陥った際の対処法（いい加減どうにかならんかな）  
[Ubuntuログインループ対処法](https://musaprg.hatenablog.com/entry/2020/06/30/201759)  
1. Ctrl+Alt+F2を押して仮想コンソールに入る  
2. `sudo vi /etc/default/grub`でgrubの設定を開く．※「x」で一文字消去．「:q!」でファイルを保存せず閉じる．「:wq」でファイルを保存し閉じる  
4. `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`から、`splash`を消す。 変更後は`GRUB_CMDLINE_LINUX_DEFAULT="quiet"`となっているのを確認  
5. `sudo update-grub`で変更を適用し，`reboot`

起動すると，`reset givinng up`のような文字が何度か表示され，最後に[OOO.OOO.OOO]のような文字が表示された後，無事起動できた．  

## VScodeの導入  
[ここ](https://code.visualstudio.com/download)からダウンロード．  
VSCodeを起動し，左部のExtensionsから「Japanese Language Pack for VSCode」をインストールし,「Restart」．  
[VSCodeの各種設定はここ](https://github.com/is0383kk/VSCode)  
VSCode拡張パッケージの「Python」「vscode-icons」「Atom One Dark」をとりあえずインストールしておく．  
「Ctrl」+「,」で設定画面を開き，右上のアイコン部分の「JSON」を開くを選択し以下を`settng.json`コピペ．  
```json
{
    "window.zoomLevel": 1,
    "editor.fontSize": 18,
    
    "workbench.iconTheme": "vscode-icons",
    "workbench.colorTheme": "Atom One Dark",
    
    "python.linting.enabled": false,

    "editor.tokenColorCustomizations": {
        "comments": {
          "foreground": "#638505"
        }
    },

    "breadcrumbs.enabled": false,
    "editor.cursorBlinking": "blink",
    "editor.hideCursorInOverviewRuler": true,
    "editor.minimap.enabled": true, 
    "editor.occurrencesHighlight": true,
    "editor.renderIndentGuides": true, 
    "editor.roundedSelection": false, 
    "editor.scrollBeyondLastLine": false,
    "explorer.decorations.colors": false,
    "explorer.openEditors.visible": 0, 
    "workbench.activityBar.visible": true, 
    "workbench.editor.showIcons": true, 
    "workbench.startupEditor": "none",
    "workbench.tree.renderIndentGuides": "none",
    "editor.insertSpaces": true,
    "editor.renderWhitespace": "all",
    "python.pythonPath": "/usr/local/bin/python3",
    "vsicons.dontShowNewVersionMessage": true,
    "explorer.confirmDelete": false,
}
```
### VSCode拡張機能  
- HTML用
    - [IntelliSense for CSS class names in HTML](https://marketplace.visualstudio.com/items?itemName=Zignd.html-css-class-completion):CSSクラスの入力補完プラグイン
    - [HTMLHint](https://marketplace.visualstudio.com/items?itemName=mkaufman.HTMLHint):HTMLの静的解析ツールです
    - [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode):ソースコードの自動整形ツール

## Java用環境構築
まずはUbuntu20.04に標準搭載されているJavaの確認．  
```
hoge:~$ java -version
openjdk version "11.0.14.1" 2022-02-08
OpenJDK Runtime Environment (build 11.0.14.1+1-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.14.1+1-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)
```

このままだと.javaファイルをコンパイルできない↓．  
```
hoge:~$ javac test.java 

コマンド 'javac' が見つかりません。次の方法でインストールできます:

sudo apt install openjdk-11-jdk-headless  # version 11.0.14.1+1-0ubuntu1~20.04, or
sudo apt install default-jdk              # version 2:1.11-72
sudo apt install openjdk-13-jdk-headless  # version 13.0.7+5-0ubuntu1~20.04
sudo apt install openjdk-16-jdk-headless  # version 16.0.1+9-1~20.04
sudo apt install openjdk-17-jdk-headless  # version 17.0.2+8-1~20.04
sudo apt install openjdk-8-jdk-headless   # version 8u312-b07-0ubuntu1~20.04
sudo apt install ecj                      # version 3.16.0-1
```

`default-jdk`をダウンロード．  
```
hoge:~$ sudo apt-get install default-jdk
```

```
hoge:~$ javac HelloWorld.java
hoge:~$ java HelloWorld.class
```

サンプル用プログラム．  
```java:HelloWorld.java
public class HelloWorld 
    {
    public static void main(String[] args)
        {
        System.out.println("Hello World.");
        }
    }
```
## HTML用環境構築

