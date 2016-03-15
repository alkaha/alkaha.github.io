---
layout: post
title: Mengaturcara Elixir - Satu Pengenalan
---

- Dokumen asal: [Programming Elixir: A Gentle Introduction](https://pragprog.com/magazines/2013-06/programming-elixir)
- Penulis asal: [Dave Thomas](https://pragprog.com)

---

Limabelas tahun lalu.  Itulah masa terakhir saya teruja dengan satu bahasa aturcara, dan bahasa itu adalah Ruby.

Bukan sebab tiada usaha.  Saya meneroka kesemua bahasa-bahasa baru semasa mereka dilancarkan, tetapi tidak ada satu pun yang menarik perhatian saya - tidak ada yang membuat saya rasa saya akan suka untuk menghabiskan masa bertahun-tahun menggunakannya.

Kemudian saya berjumpa Elixir. Dua kali.  Pertama kali adalah setahun lepas, dan masa itu ia nampak bagus, tetapi tidak menarik minat.  Tetapi Corey Haines memaksa saya untuk melihatnya kembali.  Beliau betul.  Elixir adalah istimewa.

## Cukup Dengan Hype Peminat

Maaf.

Elixir ialah satu bahasa aturcara kefungsian yang berjalan di atas 'Erlang virtual machine'.  Ia mempunyai sintaks ala Ruby, dan bercirikan 'protocols'(untuk menambah kefugnsian modul tanpa mengubah kod sumber), makro dan sokongan aturcara-meta yang bagus.  Ia juga mempunyai kebaikan-kebaikan dari pengalaman bahasa-bahasa lain, jadi ia mempunyai banyak ciri-ciri moden.  Sebagai contoh, 'protocol' dan 'macro' adalah 'lexically scoped', jadi aturcara-meta tidak lagi mempunyai risiko mengacau persekitaran perlaksanaan global.

Elixir berjalan di atas Erlang VM, Beam, dan compatible dengan kod-kod Erlang.  Ini bermakna anda dapat mengambil manafaat dari 'high availability', 'high concurrecy', dan ciri-ciri pengagihan dari Beam.  Ia juga bermakna anda boleh menggunakan beribu-ribu library Erlang, sebahagiannya dipasangdalam dan sebahagian lagi sebagai sumber pihak ketiga.  Khasnya, Elixir boleh berjalan di dalam 'OTP framework'.

## Cukup - Tunjukkan Saya Kod
(Jika anda mahu turut serta, sila muat turun Elixir.  Arahannya boleh di dapati di [laman web Elixir](http://elixir-lang.org/))

Kod di bawah menghitung jumlah kesemua elemen di dalam satu list.
{% highlight ruby %}
defmodule MyList do
	def sum([]), do: 0
	def sum([head | tail]), do: head + sum(tail)
end
IO.puts MyList.sum [1,2,3]  #=> 6
{% endhighlight %} 
Fungsi-fungsi `sum` kita berada dalam satu modul, MyList.  Fungsi itu detetapkan di dalam bentuk dua klausa.  Jadi, bagaimana Elixir tahu fungsi mana untuk dijalankan?

## Pemadanan Corak (Pattern Matching)

Ini ialah di mana pemadanan corak masuk.  List di dalam parameter fungsi pertama hanyalah satu list kosong.  Bila anda memanggil `sum`, klausa ini akan padan jika anda memanggilya bersama satu list kosong([]).  Jika berjaya dipadankan, fungsi itu memulangkan `0`.

Kalusa kedua mempuyai list parameter yang lebih rumit: [head \| tail].  Ini adalah corak yang memadankan satu list.  List tersebut hendaklah mempunyai satu elemen pertama(`head`) dan satu `tail`(baki elemen di dalam list tersebut).  Jika list yang dihantar hanya mempunyai satu elemen, elemen itu akan ditetapkan sebagai `head`, dan satu list kosogn sebagai `tail`.

Jadi mari kita panggil `MyList.sum` dengan argumen `[1,2,3]`.  Elixir mencari klausa pertama yang mempunyai corak yang padan dengan argumen tersebut.  Klausa pertama tidak padan - argumen bukan satu list kosong - tetapi klausa kedua padan.  `head` ditetapkan kepada `1`, dan `tail` kepada `[2,3]`.  Elixir menilai badan fungsi, `head + sum(tail)`, yang bermakna fungsi `sum` dipanggil sekali lagi, secara rekursif.  Urutan panggilan fungsi nampak seperti berikut:
{% highlight bash %}
sum([1,2,3])
1 + sum([2,3])
1 + 2 + sum([3])
1 + 2 + 3 + sum([])
1 + 2 + 3 + 0
{% endhighlight %}

Pemadanan corak adalah teras kepada Elixir.  Sebetulnya, ia adalah satu-satunya cara untuk mengaitkan satu nilai kepada satu pembolehubah.  Apabila kita menulis apa yang nampak seperti satu penetapan:
{% highlight bash %}
a = [1,2,3]
{% endhighlight %}
Elixir akan sibuk mencari jalan untuk memadankan bahagian kiri dengan bahagian kanan.  Di dalam kes ini, dengan mengaitkan list `[1,2,3]` kepada pembolehubah `a`.

Setelah `a` mempunyai nilai, anda boleh menulis:
{% highlight bash %}
[1,2,3] = a
{% endhighlight %}
Tidak ada masalah bagi Elixir - nilai bahagian kiri padan dengan nilai sebelah kanan.  Tapi jika anda menulis
{% highlight bash %}
99 = a
{% endhighlight %}
Elixir akan memberitahu ia tidak dapat mencari padanan.  Ia kerana Elixir hanya akan mengubah nilai yang dikaitkan dengan sesatu pembolehubah jika pembolehubah itu berada pada bahagian kiri operator padanan.

## Pemadanan Corak Data Berstruktur

Pemadanan pergi lebih jauh - Elixir akan melihat kepada struktur kedua bahagian ketika membuat padanan:
{% highlight bash %}
[a,b,c] = [1,2,3]  #a=1, b=2, c=3
[head | tail] = [1,2,3]  # head=1, tail=[2,3]
{% endhighlight %}

Sekali dikaitkan, satu pembolehubah menyimpan nilai yang sama untuk keseluruhan masa proses pemadanan corak.  Jadi, pemadanan berikut cuma akan berjaya jika pembolehubah `list` ialah satu list yang mempunyai tiga elemen dan elemen pertama dan baki elemen mempunya nilai yang sama:
{% highlight bash %}
[a, b, c] = list
{% endhighlight %}

Anda boleh mencampurkan pemalar dan pembolehubah di bahagian sebelah kiri.  Untuk menunjukkan ini, saya akan memperkenalkan satu jenis data Elixir, iaitu `tuple`.  Tuple adalah satu 'collection' nilai-nilai yang mempunyai penjang yang tetap, seperti berikut:
{% highlight bash %}
{ 1, 2, "cat"}
{ :ok, result }
{% endhighlight %}
(`:ok` di dalam baris kedua kod di atas dikenali sebagai `symbol` di dalam Elixir.  Anda boleh melihatnya sebagai pemalar string, atau sebagai 'symbol' dalam Ruby.  Anda juga boleh melihatnya sebagai satu pemalar yang nilainya ialah namanya sendiri.)

Banyak fungsi library memulangkan tuple yang mengandungi dua elemen.  Elemen pertama ialah status pulangan tersebut - jika ianya `:ok`, panggilan kepada fungsi itu berjaya; jika `:error` ianya gagal.  Elemen kedua di dalam tuple tersebut barulah hasil dari panggilan fungsi tersebut, dengan maklumat lebih banyak mengenai ralat.

Anda boleh menggunakan pemadanan corak untuk memastikan jika satu panggilan itu berjaya:
{% highlight bash %}
{ :ok, stream } = File.Open("somefile.txt")
{% endhighlight %}

Jika panggilan `File.Open` berjaya, maka `stream` akan ditetapkan sebagai hasil.  Jika tidak, pemadanan corak itu akan gagal dan Elixir akan menimbulkan ralat.(Walaubagaimanapun dalam kes ini, anda sepatutnya membuka fail tersebut dengan menggunakan `File.open!(), yang akan menimbulkan ralat yang lebih bermakna jika gagal.)

## Pemadanan Corak dan Fungsi-fungsi

Contoh `MyList.sum` kita menunjukkan pemadanan corak boleh digunakan untuk pemanggilan fungsi-fungsi.  Parameter fungsi bertindak sebagai bahagian sebelah kiri pemadanan itu, dan argumen yang dihantarkan sebagai bahagian sebelah kanan.

Berikut satu lagi contoh:  ia menghitung nilai nth nombor Fibonacci.

Kita mulakan dengan spesifikasi nombor Fibonacci:
{% highlight bash %}
fib(0) -> 1
fib(1) -> 1
fib(n) -> fib(n-2) + fib(n-1)
{% endhighlight %}

Meggunakan pemadanan corak, kita boleh menukar spesifikasi ini kepada kod yang boleh dijalankan dengan mudah:
{% highlight bash %}
defmodule Demo do
  def fib(0), do: 1
  def fib(1), do: 1
  def fib(n), do: fib(n-2) + fib(n-1)
end
{% endhighlight %}

Elixir membekalakan shell interaktif bernama `iex`.  Ini akan membenarkan kita bermain dengan fungsi `fib`:
{% highlight bash %}
$ iex fib.ex
Interactive Elixir ( ... )
iex(1)> Demo.fib(0)
1
iex(2)> Demo.fib(1)
1
iex(3)> Demo.fib(10)
89
{% endhighlight %}
Tetapi ada masalah dengan kod di atas. Apa akan berlaku jika kita memanggil `fib(-1)`?  Klausa ketiga akan berjaya dipadankan, yang aka memanggil `fib(-3)`, yang akan memanggil `fib(-5)` dan seterusnya sehingga kita kehabisan 'stack'.  Kita sepatutnya melimitkan argumen untuk `fib` hanya kepada nombor bukan negatif.

Untuk ini, di dalam Elixir, kita menggunakan klausa `guard`.  Klausa `guard` ini memberikan lebih kuasa kepada pemadanan corak dengan membenarkan kita menulis satu atau lebih 'condition' yang perlu dipadankan oleh argumen untuk pemadanan itu berjaya.  Dalam kes ini, kita boleh menulis:
{% highlight bash %}
demodule Demo do
  def fib(0), do: 1
  def fib(1), do: 1
  def fib(n) when n> 0, do: fib(n-2) + fib(n-1)
end
{% endhighlight %}
`when` di atas adalah satu klausa `guard`.  Ia mengatakan fungsi ketiga di atas cuma boleh dipadankan jika nilai argumen lebih besar atau sama dengan kosong.

Perkara yang paling menarik ialah bagaimana mudahnya untuk menterjemahkan spesifikasi ke dalam bentuk kod.  Dan setelah kod ditulis, ianya mudah untuk dibaca dan dilihat apa yang dibuat.

Perkara lain untuk diperhatikan:  Tidak ada arahan 'conditional' di dalam perlaksanaan fungsi kita( selain dari klausa guard).  Ini juga membuatkan kod tersebut senang untuk dibaca dan diselenggara.  Banyak modul-modul Elixir yang agak besar ditulis tanpa atau mengandungi amat sedikit arahan-arahan 'conditional'.

## Transformasi adalah Kerja #1

Anda mungkin berfikir semua ini amat bagus, tetapi anda tidak menulis fungsi-fungsi matematik setiap hari.

Tapi aturcara kefungsian bukan berkenaan dengan fungsi-fungsi matematik.

Fungsi-fungsi adalah perkara yang mentransformasikan data. Dalam trig fungsi `sin` mentransformasikan nilai 90 darjah kepada nilai 1.0.  Dan itulah tanda-tandanya.

Pengaturcaraan adalah bukan mengenai data.  Ia adalah berkenaan mentransformasikan data.  Setiap program yang kita tulis mengambil beberapa input dan mentransformasikannya kepada output.  Input mungkinlah satu 'web request', beberapa parameter 'command line', atau cuaca di Boise.  Apapun ia, kod kita menerimanya dan transformasikan ia beberapa kali di dalam perjalanan untuk menghasilkan keputusan yang diingini.

Dan itulah sebabnya saya fikirkan bahawa aturcara kefungsian ialah pengganti ulung kepada aturcara berasaskan objek.  Di dalam aturcara `OO`, kita selalu bimbangkan kedudukan(state) data kita.  Di dalam aturcara kefungsian, fokus kita ialah kepada kerja-kerja mentransformasikan data.  Dalam dalam proses transformasi itulah di mana nilainya ditambah.

Artikel seterusnya akan melihat kepada fungsi-fungsi, kedua-dua jenis fungsi, bernama dan tanpa nama.  Dan kita akan meneroka operator 'pipeline' yang membenarkan kita menulis seperti:
{% highlight bash %}
request |> authenticate |> lookup_data |> format_response
{% endhighlight %}

>Dave Thomas ialah seorang pengaturcara yang suka menyebarkan perkara yang hebat.  Beliau adalah salah seorang penulis buku Pragmatic Programmer, dan salah seorang pencipta Agile Manifesto.  Buku beliau Programming Ruby memperkenalkan bahasa Ruby kepada dunia, Agile Web Development with Rails membantu memulakan revolusi Rails. 







