---
layout: post
title:  "Berkenalan dengan Scala"
date:   2018-06-21 10:44:16 +0700
categories: bigdata
fbcomments: true
tags: [bigdata, hadoop, scala, spark]

---

Bermain dengan big data (hadoop), maka kita harus tahu salah satu dari bahasa pemograman berikut: Java, Scala, R, Python. Saya sendiri sebelumnya adalah Java Developer. Namun untuk big data saya lebih memilih Scala. Mengapa bukan Java? Karena Scala jauh lebih ringkas (less of code), type-safe way, mendukung functional dan OOP, dll. Lalu Phyton dan R? Saya pribadi tetap ingin tahu 2 bahasa ini. Tapi saat ini masih belajar scala dulu karena lebih mudah dipahami untuk yang sebelumnya adalah java developer.

Sebelumnya silakan install Spark & Scala terlebih dahulu di host machine Anda. Buat yang menggunakan OSX bisa mengikuti instruksi [berikut](https://medium.freecodecamp.org/installing-scala-and-apache-spark-on-mac-os-837ae57d283f)

Atau kalau malas install dan hanya ingin coba-coba, silakan gunakan <!--more-->compiler [berikut](https://scalafiddle.io)

Yang ingin saya tuliskan disini hanya seputar informasi dasar mengenai scala.
1.	tidak perlu titik koma/semicolon (;)
2.	data type tidak perlu di-definisikan, kecuali karena pertimbangan tertentu
	Contoh:
	```
	a =  10 //data type variable a otomatis di set menjadi Int
	a: Short = 10 //data type variable a kita set manual dengan Short
	```
3.	`var` (variable) & `val` (values). Nilai val tidak bisa di-re-assign/ubah (immutable), sedangkan var bisa. 
	Contoh:
	```
	var a:Int = 10
	a = 20 //tidak error
	```
	```
	val a:Int = 10
	a = 20 //error
	```
3.	Blocks,
	Contoh:
	```
	println({
	  val x = 1 + 1
	  x + 1
	}) // 3
	```
4.	Functions, di scala terdapat perbedaan antara functions dan methods
	Contoh (function sebagai value)
	```
	val addOne = (x: Int) => x + 1
	println(addOne(1)) // 2
	```
	Atau untuk yang multiple parameter seperti di bawah
	```
	val add = (x: Int, y: Int) => x + y
	println(add(1, 2)) // 3
	```
5.	Methods, ciri-ciri method: diawali dengan `def`, terdapat nama method, parameter, return type, lalu body 
	*bandingkan dengan method/functions di Java misalnya, jauh lebih simple scala kan ya*
	Contoh, pada fungsi ini kita mendefinisikan data type output-nya yaitu dalam hal ini Int.
	```
	def add(x: Int, y: Int): Int = x + y
	println(add(1, 2)) // 3
	```
	Method di scala juga bisa menerima list of parameter
	```
	def addThenMultiply(x: Int, y: Int)(multiplier: Int): Int = (x + y) * multiplier
	println(addThenMultiply(1, 2)(3)) // 9
	```
6.	Masih terdapat hal-hal lain yang di scala ada di Java tidak, misalnya case class, dll

Bagaimana? Jauh lebih menarik scala dibandingkan dengan Java (misalnya) kan? Lebih detail mengenai masing-masing topik akan dibahas di part masing-masing topik.

