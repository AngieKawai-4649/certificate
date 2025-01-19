Visual studio 2022 を使用した自己署名 (FSUSB2N SANPAKUN)

以下、SANPAKUNを例に記述するがFSUSB2Nはそれぞれの箇所を読み替えること

１．Visual Studio 2022 コミュニティ(C++によるデスクトップ開発)のインストール

    https://visualstudio.microsoft.com/ja/vs/community/
２．Windows Driver Kit のインストール

    https://learn.microsoft.com/ja-jp/windows-hardware/drivers/download-the-wdk

上記により

makecert.exe

pvk2pfx.exe

Inf2Cat.exe

signtool.exe

が使用可能となる

３．作業フォルダを作成し、sanpakun.infを入れる

４．コマンドプロンプトを起動

  スタート->Visual Studio 2022->Developer Command Prompt VS2022 を起動


５．「信頼されたルート証明機関」の証明書作成

    makecert -n "CN=SANPAKUN Root CA,O=SANPAKUN Root CA,C=JP" -a sha256 -b 01/01/2001 -e 01/01/2100 -r -pe -sv root.pvk -cy authority -eku 1.3.6.1.5.5.7.3.3 root.cer

６．「信頼された発行元」の証明書を作成

    makecert -n "CN=SANPAKUN" -a sha256 -b 01/01/2001 -e 01/01/2100 -iv root.pvk -ic root.cer -sv trustedpub.pvk -cy end -eku 1.3.6.1.5.5.7.3.3 trustedpub.cer

[オプションの説明]

-nオプションの後ろは署名の発行者情報として表示される内容。なんでもよい。

パスワードのダイアログが出るので全部123と入力

-eku オブジェクト識別子 (OID) を指定

1.3.6.1.5.5.7.3.1 サーバー認証

1.3.6.1.5.5.7.3.2 クライアント認証

1.3.6.1.5.5.7.3.3 コード署名に利用

-cy  証明機関かどうか

authority  証明機関の場合

end        エンド エンティティ (ソフトウェア発行元など)

７．署名用のファイル作成

    pvk2pfx -pvk trustedpub.pvk -spc trustedpub.cer -pfx trustedpub.pfx -f -pi 123

８．infからcatを作成

    Inf2Cat /driver:c:\driver /uselocaltime /os:7_X86,7_X64,8_X86,8_X64,6_3_X86,6_3_X64,10_X86,10_X64,Server2008R2_X64,Server8_X64,Server6_3_X64,Server10_X64,Server2016_X64

例：作業フォルダが D:\cert の場合

    Inf2Cat /driver:"D:\cert" /uselocaltime /os:7_X86,7_X64,8_X86,8_X64,6_3_X86,6_3_X64,10_X86,10_X64,Server2008R2_X64,Server8_X64,Server6_3_X64,Server10_X64,Server2016_X64

９．catに署名

    signtool sign /f trustedpub.pfx /p 123 /td SHA256 /tr http://timestamp.digicert.com /fd SHA256 sanpakun.cat

例：作業フォルダが D:\cert の場合

    signtool sign /debug /f "D:\cert\trustedpub.pfx" /p 123 /td SHA256 /tr http://timestamp.digicert.com /fd SHA256 "D:\cert\sanpakun.cat"

[重要]

同じ名前の証明書を既にインストールしていると署名に失敗するので前もって証明書を削除しておくこと

※証明書の確認/削除等

  ファイル名を指定して実行でcertmgr.mscを起動する


■コマンドによるドライバのインストール方法 (Windows標準コマンド)

必要なファイル root.cer、trustedpub.cer、driver.inf、driver.cat

certutil -addstore ROOT root.cer

certutil -addstore TrustedPublisher trustedpub.cer

pnputil -i -a driver.inf
