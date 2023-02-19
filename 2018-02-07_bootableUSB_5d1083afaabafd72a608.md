<!--
title:   ブータブルUSBを元のUSBメモリに戻す
tags:    bootableUSB
id:      5d1083afaabafd72a608
private: false
-->
# ブータブルUSBをUSBメモリとして使えるように戻すには

増設したメモリの確認用に[MemTest86](https://www.memtest86.com/)のブータブルUSBを作成したのですが、USBブートで使用した領域が使えない状態になり、フォーマットしようとしても領域自体を認識しないので困ってしまいました。ネットで探したところ[こちら](www.eastforest.jp/esthome/memo/245)に元に戻す方法が書いてあったので紹介します。
実際に行ったところ、[こちら](www.atmarkit.co.jp/ait/articles/0812/26/news119_2.html#create)にある通りにフォーマットしなおさないと使えなかったので、そちらも後半に加えました。

# 手順

コマンドプロンプトで以下を実行します。

1. `DISKPART`を入力してEnter --> DISKPARTが実行され別のコマンドプロンプト画面が開く
2. DISKPARTのコマンドプロンプト画面にて`list disk`を入力して実行し、USBメモリのディスク番号を確認する
3. `select disk <disk#>`を入力して実行し、USBメモリを選択する
4. `clean`を入力して実行し、USBメモリを初期化する
5. `list partition`でパーティションを確認する
6. `create partition primary`を実行してUSBメモリ容量と等しいサイズのパーティションを作る
7. `list volume`でボリュームを確認する
8. `select volume <volume#>`で6で作成したUSBのボリュームを選択する（Fs=filesystemsがRAWとなっているので分かりやすいはず）
9. `format fs=fat32 label="<new usb name>" quick`を実行してFAT32にフォーマットする
10. `exit`を入力して実行しDISKPARTを終了する