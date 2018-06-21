---
layout: post
title:  "Berkenalan dengan Big Data"
date:   2018-06-13 02:12:16 +0700
categories: bigdata
fbcomments: true
tags: [bigdata, hadoop, cloudera]

---

![image-title-here](/images/big_data_image.jpg){:class="img-responsive"}

Apa itu *big data*? Big itu besar, data itu ya data. Data yang besar. Pengertian sampahnya kira-kira begitulah.

Biar ga sampah-sampah banget, kita bahas sedikit lebih jauh. 
Terdapat 3 karakteristik utama big data (biasanya disebut three Vs), dan 3 karakteristik tambahan. Kita bahas satu-satu

### Volume

Volume data yang disimpan pada sistem big data tentu adalah data dalam volume besar. Tidak hanya disimpan, <!--more-->data dengan volume yang besar tersebut juga harus bisa diakses, diproses, atau diolah dalam waktu yang singkat. Sedikit sharing untuk perbandingan, di tempat kerja saya sebelumnya, traditional database hanya bisa/difungsikan untuk menyimpan 15 hari data harian. Jika ingin mengakses data 16 hari kebelakang, maka team datawarehouse harus restore dari external tape/disk terlebih dahulu. Dan secara logical, cara penyimpanan di traditional database tersebut adalah 1 database per hari. Sehingga misalkan kita punya aplikasi yang ingin menganalisa data 15 hari, maka di-aplikasi tersebut kita harus create minimal 15 db-connection.
Sedangkan setelah menggunakan big data (Hadoop), kita mampu menyimpan 10 tahun data bulanan ditambah 2 tahun data harian, belum termasuk data lain. Saya tidak bisa menyebutkan ukurannya secara angka, karena sudah bersifat confidential dan karena saya juga sudah lupa persis-nya diangka berapa :) Data tersebut masih bisa bertambah, dan ketika space sudah habis, yang perlu dilakukan adalah menambah data-node & namenode -> jika dibutuhkan (*akan dibahas kemudian*)
Dan semua data tersebut disimpan di 1 logical database. Seperti apa detailnya bisa disimpan disatu logical database, dan performance tetap OK akan saya bahas ditulisan berikutnya.
Intinya, di pengalaman tersebut perbedaan antara traditional database dan sistem big data sangat jauh. Belum lagi processing time-nya juga jauh lebih cepat di big data walaupun secara volume data yang disimpan sudah jauh berbeda. Tapi bukan berarti RDBMS akan digantikan oleh big data (Hadoop). Karena hadoop cocok untuk OLAP (analytical), tidak cocok untuk OLTP (transactional).


### Velocity

Seperti yang saya sampaikan di atas, bahwa processing time di big data juga harus cepat. Bahkan terdapat proses yang harus real-time, misalnya untuk case fraud detection, dll. Lebih detail mengenai processing time (timeliness) ini akan saya bahas di tulisan berikutnya.

### Variety

Format dan type data yang disimpan dan diolah di big data juga beragam, mulai dari yang structured (misalnya data dari RDBMS), semi-structured (xml, dll), dan yang unstructured (image, video, audio). FYI, hingga saat ini, saya sendiri belum pernah mengolah data yang unstructured di big data.

Terdapat 3 karakteristik lain yang lebih mengarah kepada tantangan di big data, a.l:
1.	**Veracity**, variety data dan kebenaran source data yang disimpan menyebabkan lebih sulit untuk memastikan kualitas dan kebenaran data
2.	**Variability**, kualitas data juga menjadi bervariasi sehingga perlu resource tambahan untuk mengindetifikasi, mengolah, dan memroses data yang berkualitas rendah sehingga data tersebut lebih berguna
3.	**Value**, tentu hal-hal di atas menjadi tantangan untuk menampilkan/memberikan value dari data tersebut.

Hal lain yang ingin saya bahas pada perkenalan big data ini adalah brief description of komponen big data itu sendiri.

1.	**[Hadoop](https://hadoop.apache.org/docs/current/)**, merupakan project Apache yang di dalam-nya terdapat *distributed file system* (HDFS) dan *resource manager* (YARN -> Yet Another Resource Negotiator). Menurut saya, ini adalah roh-nya di big data. Semua data di big data disimpan di HDFS.
2.	**[Hive](https://hive.apache.org)**, project Apache juga yang memfasilitasi pembacaan, penulisan, dan manage data yang terdapat di HDFS. Singkatnya, data di HDFS masih berbentuk file. Hive memungkinkan pengguna untuk membaca isi file tersebut menggunakan SQL syntax. Note, bukan berarti semua file di HDFS ya. File yang kita create & kita relasikan dengan database & table aja. *Lebih lanjut mengenai hive akan dibahas di chapter-chapter berikutnya*.
3.	**[Impala](https://impala.apache.org)**. Sama seperti hive, Impala merupakan query executor juga. Bedanya impala jauh lebih cepat khususnya dalam proses read. Tapi hive & impala ini sebenarnya tidak layak dibandingkan, karena mereka mempunya tupoksi masing-masing. Lalu mengapa Impala lebih cepat? Karena impala mempunyai daemon di semua node dan dia menggunakan fitur MPP (massively parallel processing). Parallel artinya bahwa query tersebut akan dilempar ke beberapa CPU di cluster dimana CPU ini run menggunakan dedicated memory. Sedangkan di Hive, Ketika SQL query kita jalankan, SQL query tersebut akan di-translate menjadi MapReduce job. Di hive, MapReduce job ini run on top of Yarn dan setiap stage dari map reduce tersebut akan menggunakan disk. Misalnya map job selesai, lalu dia akan tulis ke disk. Selanjutnya reducer akan membaca dari disk tersebut lagi *(I/O intensive)*. Belum lagi jika terdapat beberapa MR untuk 1 SQL. Bahasannya sudah tidak cocok di perkenalan. Atau singkatnya, gunakan Hive untuk DDL process dan Impala untuk proses read. *Lebih detail-nya akan dibahas di chapter-chapter berikutnya*.
4. **[Spark](https://spark.apache.org)**, merupakan computing framework yang menjalankan komputasi-nya secara terdistribusi, dan run on top of memory juga (default, ke disk jika memory habis). Di-develop tahun 2009 di UC Berkeley. Apa peran spark di big data? Saya bisa katakan, spark ini digunakan untuk hal processing. Spark ini terdiri dari beberapa component, seperti Spark DataFrame + SQL, Spark Streaming, Spark MLib (Machine Learning), GraphX. Dari komponen-nya sudah ketahuan, misalnya kita hendak membangun fraud detection system. Fraud detection ini tentu perlu real-time. Kita menggunakan spark-streaming untuk mendapatkan dan mengolah data secara real-time. Jika di dalam logic sistem tersebut kita perlu check ke database (hadoop), maka kita bisa gunakan spark SQL di dalam code kita, dst. Code ini bisa kita buat menggunakan Scala, Java, Phyton, atau R. Dengan menambahkan library-library spark ke dalam project kita, maka API component spark tersebut di atas dapat kita gunakan di dalam code/project kita. *Lebih lanjut mengenai spark dan contoh project-nya, cara deploy/submit, akan dibahas di chapter-chapter berikutnya*.
5. **[Kafka](https://kafka.apache.org/intro)**, merupakan message broker yang menyimpan data secara terdistribusi (replikasi). Buat yang sudah terbiasa menggunakan message broker seperti RabbitMQ, dll tentu akan lebih mudah familiar dengan kafka ini. Buat yang baru mendengar istilah message broker, message broker merupakan tempat penyimpanan data (broker, sifatnya sementara). Lalu apa bedanya dengan storage seperti database, dll? Broker ini punya fitur publish-subscibe. Kita bisa subsribe ke topic (mirip table di RDBMS) tertentu, lalu setiap ada data baru di topic tersebut maka program/aplikasi kita akan langsung dapat datanya (real-time). *Lebih lanjut mengenai kafka dan contoh project-nya, akan dibahas di chapter-chapter berikutnya*.
6. **[HBase](https://hbase.apache.org)**, merupakan database yang bersifat non-relational, distributed, scalable, bisa menampung *very large data*, dan yang paling penting dia bersifat key-value base. Simplenya, HBase itu seperti *huge* hash table yang cara penyimpanan data-nya dengan cara key-value. Maka mendesign key yang baik sangat penting dalam menggunakan HBase. Jika di RDBMS masing-masing record wajib mempunyai jumlah colomn yang sama, maka di HBase tidak harus. Terdapat istilah row key, column family, column, dll. Walaupun di Hive kita bisa insert update data, tetapi seperti yang saya sebutkan di atas bahwa hadoop tidak cocok dengan OLTP. Maka HBase ada untuk mendukung data OLTP, tapi jangan pernah bayangkan bahwa HBase ini sama seperti relational database. Jauh berbeda, makanya detailnya *akan saya bahas lebih detail di chapter-chapter berikutnya*
7. **[Yarn](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)**, resource manager yang tentunya mengatur computational resource untuk setiap execution. Misalnya client melakukan eksekusi job atau task tertentu, maka Yarn yang akan menentukan, oh node Manager kamu node Y ya, kamu akan di-eksekusi di node X, karena node A, B, C sedang sibuk, node D sedang down, dst (*node: server*). Detailnya *akan saya bahas lebih detail di chapter-chapter berikutnya*
8. **[Zookeeper](https://zookeeper.apache.org)**. Di big data banyak komponen-nya yang menggunakan nama hewan. Salah satunya adalah hadoop, yaitu gajah **mainan**. Seperti namanya, zookeeper (penjaga kebun binatang) merupakan services yang me-maintain konfigurasi dan dan men-sinkron-kan node-node (cluster) di hadoop.
9. **[Sqoop](http://sqoop.apache.org)**, merupakan komponen untuk transfering data dari source ke hadoop. Misalkan kita hendak meng-ingest RDBMS ke hadoop, maka sqoop adalah komponen yang tepat untuk hal tersebut.
10. **[Flume](https://flume.apache.org)**. Flume juga termasuk ingestion tools. Beda-nya, sqoop cocoknya digunakan untuk batch ingestion, flume untuk yang real-time walaupun flume sebenarnya juga bisa untuk batch. Flume lebih spesifik ditujukan untuk file ingestion, misalkan log file. Kita bisa stream dengan flume ke hadoop.
11.	**[Pig](https://pig.apache.org), [Oozie](http://oozie.apache.org)**. Ini dibahas di chapter berikutnya saja lah.
12. **[Cloudera](https://www.cloudera.com)** Cloudera sebenarnya bukan komponen hadoop. Dia adalah perusahaan di yang berkecimpung di hadoop. Salah satu produknya ya Cloudera itu sendiri. Cloudera (produk) ini menggabungkan semua komponen di atas ke satu bundle yang mereka sebut CDH (Cloudera Distribution Including Apache Hadoop). Bukan hanya menggabungkan, Cloudera ini sangat kompleks dan komplit. Terdapat detail cluster management, monitoring, BDR, dll. Kompetitor cloudera adalah Hortonworks, cuma sepertinya lebih terkenal, kece, dan lebih mahal Cloudera. Untuk Anda yang ingin belajar Cloudera silakan install dan baca-baca dokumentasi Cloudera [berikut](https://www.cloudera.com/downloads/quickstart_vms/5-13.html). Jangan lupa baca prerequisites-nya dulu karena spek host (laptop, PC)-nya harus mantap.

Masih terdapat komponen lain di big data, cuma terlalu panjang kalau dibahas disini semua. Karena ini topiknya perkenalan, tentu target pembacanya adalah orang-orang (developers) yang belum familiar dengan big data dan ingin belajar big data. Apakah semua komponen di atas harus dipelajari dulu untuk menjadi big data engineer? Tidak. Bagaimana harus memulai? Buat Anda yang mempunyai kesempatan diterima bekerja menjadi big data engineer, misalkan sebelumnya Anda adalah java developer dan belum mengenal big data, saran saya langsung terjun aja. Saya juga awalnya seperti itu. Belajar di tempat. Lalu buat Anda yang mau belajar dulu, untuk self improvement misalnya. Mulailah dari hadoop (hadoop aja cukup -> hadoop sebenarnya terdiri dari HDFS, MapReduce, Yarn), next-nya hive (kalau hadoop dirasa kurang), lalu spark. Yang sudah familiar dengan yang distributed system, atau non-relational DB akan lebih mudah mengenal big data. FYI, masing-masing komponen ini sebenarnya juga sangat kompleks. Misalkan hadoop, atau spark. Terdapat sangat banyak hal di dalam-nya yang perlu dipelajari. Saya juga masih newbie.

Ngomong-ngomong **big data engineer**, itu adalah salah satu role di big data. Masih terdapat role lain, seperti Chief Data Officer, Data Arhitect, Data Scientist, dll. 

*Untuk agan-agan yang sudah advanced di big data dan membaca tulisan ini, mohon review karena bisa saja terdapat penyampaian atau informasi yang kurang tepat.*

Note: *akan saya bahas lebih detail di chapter-chapter berikutnya* => utang

