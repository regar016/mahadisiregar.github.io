---
layout: post
title:  "Lahirnya Hadoop"
date:   2018-06-22 08:44:16 +0700
categories: bigdata
fbcomments: true
tags: [bigdata, hadoop]

---

Di part [Berkenalan dengan Big Data](https://mahadisiregar.com/bigdata/berkenalan-dengan-big-data/), saya sudah membahas apa itu big data dan komponen-komponennya. Salah satu-nya adalah hadoop. Pertama kita fix-kan dulu big data vs hadoop. Big data, ya sesuai topik [Berkenalan dengan Big Data](https://mahadisiregar.com/bigdata/berkenalan-dengan-big-data/) merupakan data yang besar, variatif, dan butuh processing time yang cepat. Sedangkan hadoop adalah salah satu tools untuk implementasi big data. Sekali lagi, hadoop itu salah tools untuk big data.

Ada yang lain selain hadoop? Ada, <!--more-->[HPCC Systems](https://hpccsystems.com), [Twitter/Backtype Storm](https://github.com/nathanmarz/storm), [Microsoft Azure Table, Project Daytona and LINQ](http://msdn.microsoft.com/en-us/magazine/ff796231.aspx), dll.

Doug Cutting, penemu Hadoop yang saat itu sedang bekerja di Yahoo! bertugas membuat sebuah library untuk text search yang disebut Lucene. Lucene ini tujuannya untuk search engine yang lebih cepat dan mampu melakukan indexing terhadap 1 milyar halaman web.
Doug Cutting mentok dan meng-open source-kan Lucene lalu melahirkan Apache Lucene di 2001. Mike Cafarella lalu datang membantu untuk indexing semua web pages yang ada. Lalu mereka melahirkan Apache Nutch.

Selama periode tersebut, mereka menghadapi 4 masalah:
1.	Schema less, data yg tidak ada tables dan columns
2.	Durable, data yang sudah ditulis tidak akan hilang
3.	Component failure handling (CPU, Memory, Network)
4.	Auto Balance (penggunaan disk)

Di tahun 2003, Google mempublish GFS paper yang menjelaskan asitektur Google’s distributed file system. Dengan mengkombinasikan Apache Nutch dan GFS, Doug Cutting dan Mike Cafarella melahirkan NDFS( Nutch Distributed File System). Tetapi NDFS belum bisa menjawab masalah durability dan fault tolerance. Ide muncul untuk distributed processing dan memecah file ke beberapa bagian/block (ukuran block = 64mb) dan menyimpan block-block tersebut secara replikasi ke 3 nodes. NDFS tentu butuh parallel processing karena data sudah disimpan secara terdistribusi.

Di tahun 2004, Google mempublish paper MapReduce. MapReduce menjawab masalah Parallelization, Distribution, dan Fault-tolerance. Di tahun 2005, Cutting menggabungkan MapReduce dengan NDFS. Tahun 2006, Cutting mengeluarkan GFS dan MapReduce dari NDFS dan mengganti namanya menjadi  **Hadoop**.
Hadoop lalu digunakan oleh beberapa perusahaan seperti Facebook, Twitter, LinkedIn.
Di 2008, Cutting memisahkan Hadoop dari Lucene dan melinsesikan Hadoop di Apache Software Foundation. Beberapa perusahaan mengalami masalah dengan file system-nya lalu bereksperimen dengan Hadoop dan melahirkan sub-projects seperti Hive, PIG, HBase, Zookeeper.

Bdw Hadoop adalah nama mainan anak-nya Doug Cutting. Anaknya baru berumur 2 tahun saat itu dan baru belajar berbicara. Anaknya tersebut memanggil mainannya yaitu boneka gajah kuning dengan nama Hadoop. Nama ini kemudian digunakan oleh Cutting untuk project-nya.

![doug cutting child toys](/images/hadoop-doug-toys.jpg){:class="img-responsive"}

