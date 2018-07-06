---
layout: post
title:  "Pertimbangan Penyimpanan Data di Hadoop [Compression]"
date:   2018-07-06 15:44:16 +0700
categories: bigdata
fbcomments: true
tags: [bigdata, hadoop]

---

Ketika mulai menggunakan Hadoop, salah satu yang sangat penting diperhatikan adalah bagaimana kita akan menyimpan data di hadoop. Bagaimana ini dalam artian format file yang kita gunakan apa, compression, disimpan dimana (HDFS, HBase, dll), directory HDFS akan seperti apa, dll. Mengapa ini penting? Misalkan dalam hal compression, compressi yang tepat akan mengurangi penggunaan disk. Tidak hanya disk, distributed processing identik dengan jaringan, sehingga data yang bergerak di jaringan pun lebih ringan dan lebih cepat. Biar lebih detail, kita bahas sedikit lebih jauh. <!--more-->

### Compression

Apa saja yg di-compress?

##### Compressing input files

Input file maksudnya, misalkan kita punya job yang membaca file dari hadoop. File dari hadoop tersebut adalah input file yang diolah si job tersebut. Nah, jika input file tersebut di-compress maka jumlah bytes yang dibaca dari HDFS (disk -> IO) tentu akan lebih kecil sehingga lebih cepat.

Decompress-nya dimana? Di job-nya. Ketika input file tersebut di pass ke MapReduce misalnya, maka map-reduce akan men-decompress file tersebut dengan codec yang sesuai dengan file extension-nya. Semakin kecil ukuran compresi semakin lama untuk di-decompress

##### Compressing output files
Menentukan format compresi ketika writing file to hadoop

##### Compressing map output
Setiap stage di MapReduce mem-passing data dengan menulis ke disk. Compressi seperti LZO, atau snappy misalnya akan memperkecil ukuran data yang di-pass di setiap stage-nya.

##### Format compresi di Hadoop
Berikut adalah type compresi yang ada di Hadoop

| Compression Format | Tool | Algorithm | File Extension | Splittable |
| ------ | ------ | ------ | ------ | ------ |
| gzip | gzip | DEFLATE | .gz | Tidak |
| bzip2 | bizp2 | bzip2 | .bz2 | Ya |
| LZO Drive | lzop | LZO | .lzo | Ya (jika menggunakan indexing) |
| Snappy | N/A | Snappy | .snappy | Tidak |


**gzip**

Gzip mantap dalam hal kompresi. Dibandingkan dengan snappy, size yang dihasilkan bisa 2.5kali lebih kecil. Tetapi write performance-nya bisa 2kali lebih lama dibandingkan dengan snappy. 
Ingat kembali bahwa size berhubungan dengan block size, block size berhubungan dengan jumlah task processing, jumlah task processing berhubungan dengan performance. Artinya karena gzip menghasilkan file yang lebih kecil, jumlah block-nya juga lebih kecil, sehingga jumlah task yang muncul untuk memproses data tersebut juga lebih sedikit.

**bzip2**

Hasil kompresi bzip lebih kecil lagi dibandingkan dengan gzip, 9% lebih kecil. Tapi performance-nya (write/read) 10 kali lebih lambat dibandingkan dengan gzip. Bzip ini sebenarnya kurang ideal untuk hadoop, karena salah satu component big data kan velocity (kecepatan processing) juga. Tapi jika Anda lebih mengutamakan ukuran dibandingkan dengan performance, ya cocok menggunakan Bzip dibandingkan dengan gzip, snappy, dll. Contoh case-nya untuk menyimpan semacam archive data yang jarang diakses dan dalam ukuran besar. 

**LZO**

LZO ini mirip dengan snappy, mengutamakan speed dibandingkan size. LZO ini splittable, tetapi harus ada indexing. 
Dibandingkan dengan gzip misalnya, process read LZO lebih cepat 2 kali karena decompress LZO juga 2 kali lebih cepat.

**Snappy**

Snappy ini dikembangkan oleh Google, tetapi tujuan utamanya bukan untuk menghasilkan file yang sangat kecil. Snappy lebih mengutamakan performance/speed dibadingkan size/ukuran. File kompresi snappy bisa 20% hingga 100% lebih besar

Dari 4 type compressi tersebut, hingga saat ini saya baru pernah mencoba gzip dan snappy. Dan untuk final storage, di tempat pekerjaan sebelumnya kami menggunakan snappy dengan alasan performance.

### Compression & Splitable
Apa sihhhh splittable? Misalkan file 1GB (tanpa compressi) hendak di proses. 1 block 128MB, berarti akan HDFS akan menyimpan data ini ke 8 block. Ketika file ini hendak diprosess, MapReduce yang mengolah file ini akan membuatkan 8 split (sesuai block) karena file-nya (tanpa compression) mendukung split. Dan masing-masing split akan diproses oleh map task yang berbeda.
File 1GB tadi lalu di-compress menggunakan gzip. Masih sama, HDFS akan menyimpan file ini ke 8 block. Ukuran file tentu akan lebih kecil tetapi karena gzip tidak splittable, maka data 8 block tersebut hanya akan diolah oleh 1 map task karena MapReduce-nya tidak bisa melakukan split. Processing time-nya tentu akan menjadi lebih lama.

**Lalu apa yang di-rekomendasikan mengenai compression ini?**

Sebenarnya kalau menggunakan file format Avro, SequenceFiles, dll (container file formats), ya semua compression di atas ujung-ujungnya mirip splittable juga. Karena container file formats mendukung block record compression (artinya sekumpulan record dicompress) lalu sekumpulan block ini yang nantinya akan di pas ke map task. Bahkan container file formats mendukung compression level record (per 1 record). Tetapi jika tidak disimpan dengan container file formats, makan gunakanlah bzip sehingga ketika diproses file-nya bisa di-split oleh MapReduce.

Demikian untuk saat ini. Untuk contoh code penyimpanan data menggunakan compression di atas akan dibahas di part selanjutnya, sekaligus sebagai pembuktian.
