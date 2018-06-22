---
layout: post
title:  "Berkenalan dengan Hadoop"
date:   2018-06-22 13:44:16 +0700
categories: bigdata
fbcomments: true
tags: [bigdata, hadoop, hdfs, yarn]

---

Hadoop, merupakan salah satu tools untuk big data. Kita akan bahas sedikit lebih detail mengenai hadoop di part ini.

**HDFS**

***Master Slave Architecture***

![hadoop architecture ilustration](/images/hadoop-architecture-ilustration-1.png){:class="img-responsive"}

Pada ilustrasi di atas, si John menangani project A, Doe menangani project B, dst. Misalkan si Auna sakit atau resign, maka <!--more-->project D akan terkendala. Lalu masing-masing orang diminta membackup yang lain seperti pada gambar di bawah. *Ini hanya ilustrasi ya guys, bukan utilisasi. Dan secara default & standar, data biasanya di-replikasi di 3 datanode (3 replikasi). Pada ilustrasi ini, hanya 1 replikasi*. Sehingga ketika Auna sakit atau resign, maka si Doe masih bisa memberikan laporan tentang project D.

![hadoop architecture ilustration](/images/hadoop-architecture-ilustration-2.png){:class="img-responsive"}

Ilustrasi di atas bisa dimirip-miripkan dengan Hadoop. Project Manager pada gambar tersebut bisa diilustrasikan sebagai Master Node, lalu John, Doe, dkk sebagai Slave Node. Lebih jauh, Master Node ini disebut sebagai ***NameNode*** dan Slave node disebut sebagai ***DataNode***. Lalu bagaimana jika si NameNode yang down? Maka muncullah ***Secondary NameNode***. 

![hadoop architecture](/images/hadoop-architecture-1.png){:class="img-responsive"}

Lalu apa fungsi masing-masing node tersebut?

***NameNode***
1.	Me-maintain dan me-manage datanode. 
2.	Menyimpan metadata, seperti informasi mengenai data block, lokasi block tertentu disimpan di datanode mana beserta alamat disk-nya, ukuran file, permission, dll
3.	Menerima heartbeat (detak jantung) dan block report dari masing-masing datanode. Ya mereka ini intim guys. Untuk memastikan apakah datanode tertentu berfungsi dengan baik dan apakah block data tertentu bisa digunakan (tidak corrupt, dll)
4.	Meng-handle replikasi data
5.	Failure handler, misalkan salah satu datanode down. Maka namenode bertanggung jawab untuk mengatur replikasi data (ke datanode yg mana saja akan di-replikasi), balancing, dan menghandle request terhadap data di datanode yang down tersebut. Sehingga tetap bisa diakses/dibaca oleh client.

***DataNode***
1.	Menyimpan data
2.	Menghandle read & write request dari client

***Secondary NameNode***
Namenode merupakan master-nya di HDFS. Sekalinya namenode rusak, katakanlah namenode down, disk-nya rusak dan tidak bisa di-recover, ya wes. Itu artinya sama aja semua data/file di datanode tidak bisa digunakan lagi karena detail data/file tersebut ada di namenode. Sehingga namenode perlu *high availability*. Lalu apa secondary namenode? Merupakan backup dari NameNode? Bukan 100% backup. Cara kerjanya seperti ini:
1.	Untuk meng-handle metadata, namenode menyimpan 2 files yaitu **FsImage** dan **EditLogs**
2.	FsImages dan EditLogs ini gunanya untuk menyimpan semua perubahan yang terjadi di HDFS. FsImages untuk menyimpan perubahan yang terjadi > 1 jam dari sekarang (default) & disimpan di disk. Sedangkan EditLogs menyimpan perubahan yang baru terjadi (<= 1 jam dari sekarang) dan disimpan di memory.
3.	Di Secondary NameNode juga terdapat fsImages dan secara periodic akan melakukan query/download EditLogs (perubahan yang terjadi) dari NameNode
4.	Secondary NameNode lalu menggabungkan FsImages & EditLogs. Hal ini disebut dengan istilah **Checkpointing**. Selama checkpointing berlangsung, perubahan yang terjadi di namenode akan disimpan di file EditLogs yang baru.
3.	FsImages yang baru dari secondary namenode akan di-copy kembali ke namenode untuk menjaga metadata di namenode tetap update
4.	Secondary namenode menangani checkpointing sehingga namenode tidak terganggu. Checkpointing mencegah file EditLogs membengkak sehingga memory juga tidak ikut membengkak. Impactnya, Failover bisa ditangani dengan cepat.

***Bagaimana Data Disimpan di DataNode?***

Setiap file disimpan sebagai block di HDFS. Ukuran default block adalah 128MB (di Apache Hadoop 2.x), atau 64MB di Hadoop 1.x. Misalnya client/kita ingin menyimpan data/file sebesar 380MB di HDFS. File ini akan dipecah menjadi 3 blok (128MB - 128MB - 124MB). Lalu misalnya block pertama akan disimpan di datanode-A, block kedua disimpan di datanode-B, dan block ketiga disimpan di datanode-C. Yang perlu kita ketahui adalah 
1.	block akan disimpan di datanode mana ditentukan oleh namenode, sesuai heartbeat & balance. demikian juga untuk replikasi dari masing-masing block
2.	setiap block mempunyai ukuran 128MB. Jika block ketiga pada kasus di atas hanya 124MB, maka block ini akan tetap mengkonumsi disk sebesar 128MB, bukan 124MB. 4MB-nya akan terbuang. Ga masalah lah, 4MB doang (kali 3 karena replikasi). Tapi bagaimana jika block yang kita simpan mencapai ribuan atau jutaan block? Maka block-size ini juga perlu diperhatikan
3.	bagaimana jika file yang kita simpan adalah file-file kecil? Misalnya 10MB. 10MB tetap membutuhkan 1 block size, yaitu 128MB. Artinya akan semakin banyak disk yang terbuang. Bukan hanya itu, namenode juga akan membengkak karena harus menyimpan banyak metadata untuk file-file kecil tersebut. Di hadoop, masalah ini disebut dengan istilah **small-files** dimana kita banyak menyimpan file-file kecil di HDFS kita. Padahal hadoop itu sendiri ditujukan untuk large files (volumes).
4.	*atau untuk menangani masalah point 3 di atas, kita kecilkan ukuran block size-nya. Misalnya menjadi 30MB per block size*. Ini bukan solusi yang bagus juga karena, misalkan kita akan menyimpan file berukuran besar, 10GB. Akibatnya file ini akan disimpan di banyak block dan lagi-lagi akan memperberat namenode. Tidak kena small-files memang, tapi akan mempengaruhi processing time. Misalnya file 10GB ini akan diolah, lalu kemudian process-nya akan mendapat banyak alamat dari namenode. Melakukan pembacaan ke disk untuk setiap alamat tersebut lalu distributed processing-nya juga akan semakin rempong. Karena akan banyak process yang mengolah data dengan block kecil, lalu masing-masing process akan nge-sync. Tentu ini tidak baik.

Lalu bagaimana bagusnya? sesuaikan ukuran block size dengan ukuran file rata2 Anda. 128MB sudah termasuk standar, jika file-file Anda kecil2, mungkin perlu merging, dll. Tapi merging juga tidak bisa serta merta langsung merge, karena terkait masalah partitioning, bucketing, dll. Maka lebih lanjut mengenai hal ini akan dibahas dikemudian hari.

**YARN**

*menjelaskan lebih detail tentang Yarn, Node Manager, dll*

***Bagaimana Proses Penyimpanan Data di Hadoop?***

*menjelaskan lebih detail mulai dari client menjalankan commit put, atau copy, dll hingga data tersimpan di HDFS*

