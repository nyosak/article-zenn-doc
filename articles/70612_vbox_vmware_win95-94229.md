---
title: "VirtualBox で Windows 95 を久々に起動する"
topics: ["VirtualBox", "Win95", "VMware", "Ubuntu", "ShiftJIS"]
type: "tech"
emoji: "🐚"
published: true
---
# VirtualBox で Windows 95 を久々に起動する

30年前に発売されたデスクトップOSは今でも役に立つのか。

---

# 🌒️ 序

Windows 95 日本語版。こいつがどうしても必要になった。

どうしても動かしたいニッチ系ソフト（バイナリ実行形式）があって、 Windows 95/98 かつ日本語という環境が要求されている。ちなみに、 win32 api で動いているので Windows 10 とか 11 でも起動はする。ただ、日本語表示のダイアログ等が文字化けして読めない。おそらく 16 bit DOS と親和性のある codepage 932 (Shift_JIS) に固定されたライブラリに依存しているのだろう。

# 🌕️ 破

## 以前に VMware で使っていたイメージが残っていた

バックアップを取った CD-R を掘り当てた。VMware の vmdk イメージは VirtualBox で読み込めたはず。

## VirtualBox のインストール

Ubuntu 22.04 に導入する。

https://www.virtualbox.org/wiki/Linux_Downloads

公式ページに、 Ubuntu 22.04 用 deb パッケージへのリンクがあるが、こいつは無視して、少し下にスクロールし、 `Debian-based Linux distributions` のところを見る。

`/etc/apt/sources.list` に追記せよとある部分を、より好ましい、 `/etc/apt/sources.list.d/` に専用ファイルを作る方式に変更して実施する。

```bash
$ cat /etc/apt/sources.list.d/virtualbox.list
deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian jammy contrib

```

```bash
$ wget -O ~/Downloads/oracle_vbox_2016.asc https://www.virtualbox.org/download/oracle_vbox_2016.asc

$ gpg --dry-run --import --import-options show-only ~/Downloads/oracle_vbox_2016.asc
pub   rsa4096 2016-04-22 [SC]
      B9F8D658297AF3EFC18D5CDFA2F683C52980AECF
uid                      Oracle Corporation (VirtualBox archive signing key) <info@virtualbox.org>
sub   rsa4096 2016-04-22 [E]

```

公式にある fingerprint と合致した。

> B9F8 D658 297A F3EF C18D  5CDF A2F6 83C5 2980 AECF
> Oracle Corporation (VirtualBox archive signing key) <info@virtualbox.org>

私的に追加する keyring の格納場所は、 `/usr/share/keyrings/` でなく `/etc/apt/keyrings/` が好ましいようなので、

```bash
$ sudo gpg --yes --output /etc/apt/keyrings/oracle-virtualbox-2016.gpg --dearmor ~/Downloads/oracle_vbox_2016.asc
```

```bash
sudo apt update
```

> Err:11 https://download.virtualbox.org/virtualbox/debian jammy InRelease
>   The following signatures couldn't be verified because the public key is not available: NO_PUBKEY A2F683C52980AECF

あれ、エラーになったぞ。
そうだ。最初に作ったファイルにも keyrings のパスが書かれているので、そちらも変更だ。

```bash
$ cat /etc/apt/sources.list.d/virtualbox.list
deb [arch=amd64 signed-by=/etc/apt/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian jammy contrib

```

> Get:13 https://download.virtualbox.org/virtualbox/debian jammy/contrib amd64 Packages [1,956 B]
> Fetched 10.3 kB in 2s (4,464 B/s)

```bash
sudo apt install virtualbox-7.1
```

```
vboxdrv.sh: failed: Look at /var/log/vbox-setup.log to find out what went wrong.

There were problems setting up VirtualBox.  To re-start the set-up process, run
  /sbin/vboxconfig
as root.  If your system is using EFI Secure Boot you may need to sign the
kernel modules (vboxdrv, vboxnetflt, vboxnetadp, vboxpci) before you can load
them. Please see your Linux system's documentation for more information.

```

初期化でエラーが起きたようだ。

```bash
$ cat /var/log/vbox-setup.log
Building the main VirtualBox module.
Error building the module:
make V=1 CONFIG_MODULE_SIG= CONFIG_MODULE_SIG_ALL= -C /lib/modules/6.8.0-60-generic/build M=/tmp/vbox.0 SRCROOT=/tmp/vbox.0 -j8 modules
make[1]: warning: -j8 forced in submake: resetting jobserver mode.
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-12 (Ubuntu 12.3.0-1ubuntu1~22.04) 12.3.0
  You are using:

... to be continued
```

おそらく、 host 側と client 側で、ハードウェアの直接共有とかをやる時に必要となる module を host 側で稼働させないといけないのだが、こいつを kernel に合わせてビルドしないといけない。今どきの Ubuntu で kernel を自前でビルドしている人は少数派なので、 kernel build できる環境は普通持っていない。こいつはかなり無理な要求をしている。

サーバーにしたいだけなら、こんなもの無くても動くはずだが、今は GUI な OS を起動しようとしているので、無いと困るかも。

Mac版とかだと、どこかに、OSに合わせてビルド済みの module が転がっていたが、 Linux はどうなんだろう？

## とりあえず起動してみる

![cannot_enumerate_usb.png](https://nyosak.github.io/article-base-doc/media/70612_vbox_vmware_win95_cannot_enumerate_usb.png)

> Can't enumerate USB devices ...

予想したとおりだ。だが、これで動くならその選択肢もあるのか。とりあえず、解決策を探ってみる。

- [How to fix and prevent VirtualBox Kernel driver not installed](https://superuser.com/questions/1285964/how-to-fix-and-prevent-virtualbox-kernel-driver-not-installed)

こいつの 5番の答えを参考にした。

```bash
sudo apt install libelf-dev
sudo apt install virtualbox-dkms
```

`Dynamic Kernel Module Support (DKMS)`

ベースになるのは古い6系バージョンのようだが、エラーが出ない。これで進めてみる。

## 久々に CD を読む

ミニPC 使っているので、本体サイズが CD のメディアより小さい。当然、ドライブなど内蔵していない。外付けがどこかにしまわれていたはず。。。

USB接続6倍速CDドライブ なるものが出てきた。むっちゃ遅いやつだ。 USB A-B タイプのケーブル。極性と電圧が適合するDCアダプタ。これで役者は揃った。

ダメだ。壊れそうなモーター音がするだけで、マウントできない。

ちなみに、このドライブは日本製だ。日本製のPCが世界で優位だった時代はCDではなくFDが主役だったから、日本がPCハードウェアの主役から降りた後の製品になる。壊れないという得意技まで捨ててしまったのだから、世界の市場で生き残れなかったわけだ。

作戦変更。内蔵ドライブを持った古いデスクトップを起動してみる。読めたので、USBドライブに転送する。

## vmdk

作ってみる。

`エラー。このまま停止した`

![guru_meditation.png](https://nyosak.github.io/article-base-doc/media/70612_vbox_vmware_win95_guru_meditation.png)

![windows95_splash.png](https://nyosak.github.io/article-base-doc/media/70612_vbox_vmware_win95_windows95_splash.png)

余計なusbとか全部無効にする。

`エラー（画像略）`

調べてみると、こいつが必要らしい。

- [Windows 95 High-Speed Processor Support v. 3.0 By LoneCrusader](https://archive.org/details/fix-95-cpu-v3-final)

最近の cpu clock は当時から見れば想像できない速さなのでエラーとみなされるらしい。

いちおう、中身を確認しておく。

```bash
sudo losetup -f --show -P /opt/vboxes/w95j/FIX95CPU.IMA
sudo mount /dev/loop32 /mnt
ls /mnt
cat /mnt/AUTOEXEC.BAT
cat /mnt/FIX95CPU.BAT
sudo umount /mnt
sudo losetup -d /dev/loop32
```

中身のダイジェスト版はこんな感じ。

```bat
IF EXIST C:\MSDOS.SYS GOTO DETECT
ECHO                       You must install Windows 95 to C:\.
GOTO QUIT

:DETECT
FIND.EXE < C:\MSDOS.SYS > C:\!W95DIR.ARG /I "WINDIR"
COPY A:\SET.ARG + C:\!W95DIR.ARG C:\!W95VARS.BAT
CALL C:\!W95VARS.BAT

GR /V:0 /1234 C:\!W95VARS.BAT "WinDir=" "RegDir="
GR /V:0 /1234 C:\!W95VARS.BAT \ \\
CALL C:\!W95VARS.BAT

DEL C:\!W95DIR.ARG
DEL C:\!W95VARS.BAT

IF NOT EXIST %WINDIR%\WUPDMGR.EXE GOTO W95
ECHO                       This update is for Windows 95 only.
GOTO QUIT

:W95
ECHO Copying new files...
COPY /Y A:\W95BOTH\SYSTEM\*.* %WINDIR%\SYSTEM
COPY /Y A:\W95BOTH\SYSTEM\IOSUBSYS\*.* %WINDIR%\SYSTEM\IOSUBSYS
COPY /Y A:\W95BOTH\SYSTEM\VMM32\*.* %WINDIR%\SYSTEM\VMM32
IF EXIST %WINDIR%\FILEXFER.EXE GOTO OSR2
COPY /Y A:\W95RTM\*.* %WINDIR%
COPY /Y A:\W95RTM\SYSTEM\IOSUBSYS\*.* %WINDIR%\SYSTEM\IOSUBSYS
ECHO Updating system registry...
COPY A:\W95RTM.REG %WINDIR%\FIX95CPU.REG
GR /V:0 /1234 %WINDIR%\FIX95CPU.REG "C:\\WINDOWS" %REGDIR%
COPY %WINDIR%\REGEDIT.EXE %WINDIR%\REG95RTM.EXE
CALL %WINDIR%\REG95RTM.EXE /L:%WINDIR%\SYSTEM.DAT /R:%WINDIR%\USER.DAT %WINDIR%\FIX95CPU.REG
DEL %WINDIR%\REG95RTM.EXE
GOTO W95POST

:OSR2
COPY /Y A:\W95OSR2\SYSTEM\IOSUBSYS\*.* %WINDIR%\SYSTEM\IOSUBSYS
COPY /Y A:\W95OSR2\SYSTEM\VMM32\*.* %WINDIR%\SYSTEM\VMM32
ECHO Updating system registry...
COPY A:\W95OSR2.REG %WINDIR%\FIX95CPU.REG
GR /V:0 /1234 %WINDIR%\FIX95CPU.REG "C:\\WINDOWS" %REGDIR%
CALL %WINDIR%\REGEDIT.EXE /L:%WINDIR%\SYSTEM.DAT /R:%WINDIR%\USER.DAT %WINDIR%\FIX95CPU.REG

:W95POST
DEL %WINDIR%\FIX95CPU.REG
%WINDIR%\WININIT.EXE
COPY /Y A:\W95BOTH\SYSTEM\VMM32\POST\*.* %WINDIR%\SYSTEM\VMM32

:QUIT

```

Windows 95 を C ドライブに導入している必要があるが、インストール先ディレクトリが必ずしも Windows でなくても、自動判定する仕組みが入っている。

導入はうまくいった。

起動はできない。

`エラー（画像略）`

使用しているのは AMD Rythen だ。
こいつ特有の仮想化問題があるのか？

仮想系の設定を、なるべくハードウェア依存しない方式にしてみる。

- Acceleration NONE
- disable Enable nested paging

`エラー（画像略）`

cpu を fake してみる

```bash
VBoxManage modifyvm "w95j_vs6" --cpu-profile "Intel Core i5-3570"

```

`エラー（画像略）`

他のOSのイメージとかを試してみる。

- DOS は起動した。
- Windows 2000 は起動しない。
- 試しに、Windows 95 起動中に F8 キーで割り込み、 Command Prompt 起動してみたら、普通に起動した。

Windows 95 も Windows 2000 も起動スプラッシュが出た瞬間に落ちる。タイミングが一致するので同じ原因の可能性がある。 CPU mode を 16 bit 互換から 32 bit native に切り替える時に落ちるのだとしたら、やはり AMD Rythen の問題かも。

じゃあ、新しい方を使うといいのかも。

7.1 に戻ってみる。

```bash
sudo apt remove virtualbox-dkms
sudo apt install virtualbox-7.1
```

うまくいったように見える。

```bash
$ sudo /sbin/vboxconfig
vboxdrv.sh: Stopping VirtualBox services.
vboxdrv.sh: Starting VirtualBox services.
vboxdrv.sh: Building VirtualBox kernel modules.

```

module が build できた。
どうやら、 `libelf-dev` がすべてを解決したようだ。

作った vm は新しいバージョンにそのまま引き継がれている。

Windows 95 が起動した。

VGAドライバが非対応なので、次のドライバを導入する必要がある。

- https://web.archive.org/web/20190210203844/http://bearwindows.boot-land.net/140214.zip

CD がマウントできず、導入できない。 VirtualBox Guest Additions も導入できない。
そういえば、先駆者のみなさん、最初にすべてをハードディスクに転送してから setup に進むという手順だった。
そういうことか。

1. Windows 2000 だとCDをマウントできるので、まずはそちらに VirtualBox Guest Additions を導入した。これで、host 側とファイル共有できるようになり、作業可能になる。
2. Windows 2000 側に、 Windows 95 の C ドライブを一時的にマウントして、 Windows 2000 経由で必要なファイルを転送する。
3. Windows 95 を起動して、ディスプレイ設定を調整する。

以上の手順で、とりあえずは起動した。
ハードウェアの不整合があるっぽくて、何やらドライバを導入しようとするが、インストールディスクを挿入できない以上、何もできない。
また、 Guest Additions は、あいかわらず導入できない。/usr/share/virtualbox/VBoxGuestAdditions.iso のイメージを使って、ハードディスクから起動してもエラーになる。subst でドライブレターを割り当ててみても結果は変わらなかった。

![guest_addition_error.png](https://nyosak.github.io/article-base-doc/media/70612_vbox_vmware_win95_guest_addition_error.png)

それでも、目的のソフトは動いた。
Guest Additions 入れてないので、マウスの扱いに神経が必要だが、使えた。

![band18_857.png](https://nyosak.github.io/article-base-doc/media/70612_vbox_vmware_win95_band18_857.png)


最初に出ていた usb enumeration error は次の手順で解決した。

```bash
sudo usermod -a -G vboxusers $USER
```


# 🌖️ 急

うまくいかず、余計な回り道をした部分が多かった。しかし最終的に、目的は達成できた。インターネットに蓄積された先人たちの知恵の偉大さをあらためて実感した。（インターネット・アーカイブにしか保存されていないファイルリンクにも頼ったし）

FIX95CPU の作者が記した言葉を引用して、ひとまずの区切りとする。

> Long Live Windows 95! Good Luck!


## 参考資料

- [Oracle VirtualBox](https://www.virtualbox.org/)
- [Download VirtualBox for Linux Hosts](https://www.virtualbox.org/wiki/Linux_Downloads)
- [How to fix and prevent VirtualBox Kernel driver not installed](https://superuser.com/questions/1285964/how-to-fix-and-prevent-virtualbox-kernel-driver-not-installed)
- [Windows 95 High-Speed Processor Support v. 3.0 By LoneCrusader](https://archive.org/details/fix-95-cpu-v3-final)
- [インターネット・アーカイブ上の http://bearwindows.boot-land.net/140214.zip 2019-2-10 時点](https://web.archive.org/web/20190210203844/http://bearwindows.boot-land.net/140214.zip)
- [今のパソコン上のVirtualBoxにWindows95を入れる](https://qiita.com/mikuta0407/items/fd7cc8bdf3153c7f97b9)
- [【備忘録】Virtualbox仮想マシンへのWindows95 OSR1の構築](http://kykyblog.air-nifty.com/blog/2021/09/post-d15ee7.html)
