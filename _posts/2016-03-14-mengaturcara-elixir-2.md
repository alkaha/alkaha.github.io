---
layout: post
title: Mengaturcara Elixir Bahagian 2 - Fungsi dan Pipeline
---

- Dokumen asal: [Programming Elixir: Functions dan Pipeline](https://pragprog.com/magazines/2013-07/programming-elixir)
- Penulis asal: [Dave Thomas](https://pragprog.com)

---

>Elixir ia satu bahasa aturcara kefungsian moden direkacipta untuk ketahanan tinggi(high availability) dan konkurensi(concurrency).  Ia mempunya sintaks ala RUby dikahwinkan dengan kuasa dan ketahanan Erlang VM.  Jika anda mahu bermula dengan aturcara kefungsian tetapi tidak suka dengan rasa akademik, sekarang adalah masa untuk terjun ke dalamnya.

[Di dalam artikel lepas](http://alkaha.github.io/mengaturcara-elixir-1), kita telah melihat asas pemadanan corak dan bagaimana ianya terdapat di mana-mana di dalam Elixir - ia adalah satu-satunya cara untuk memegang satu nilai kepada pembolehubah atau parameter.  Tetapi pemadanan corak betul-betul bersinar apabila kita menggunakanya di dalam fungsi-fungsi.

## Fungsi Tanpa nama (Anonymous Function)

Elixir mempunyai beberapa set modul terbina dalam yang bagus.  Salah satunya, Enum, membenarkan anda bekerja menggunakan 'enumerable collections'.  Salah satu fungsi lazim adalah 'map', yang mana mengaplikasikan fungsi transformasi kepada satu 'collection', dan menghasilkan satu 'collection' baru.  Jalankan shell interaktif Elixir, `iex`, dan mencubanya.
{% highlight bash %}
iex> Enum.map [2,4,6], fn val -> val * val end
[4,16,36]
{% endhighlight %}

Argumen pertama yang kita hantar kepada 'map' ialah 'collection'; di dalam kes ini satu senarai tiga nombor bulat.  Argumen kedua ialah satu fungsi tanpa nama.

Fungsi tanpa nama(saya akan memanggil mereka `fn` dari sekarang) ditetapkan diantara katakunci 'fn' dan 'end'.  Satu panah menghadap ke kanan, '->' membahagikan satu senarai kosong atau lebih parameter di bahagian kiri dari badan fungsi di sebelah kanan.

Kita menghantar 'map' fungsi `fn val -> val * val end`.  Fn ini mengambil satu parameter, `val` dan badan fungsi mendarab nilai itu dengan dirinya sendiri, untuk memulangkan hasil.

Satu fn adalah hanya satu nilai ELixir, jadi kita juga boleh menulis kod tersebut sebagai:
{% highlight bash %}
iex> square = fn val -> val * val end
#Function<erl_eval.6.17052888>
iex> Enum.map [2,4,6], square
[4,16,36]
{% endhighlight %}

Anda boleh memanggil fn sebagaimana memanggil lain-lain fungsi biasa.
{% highlight bash %}
iex> square.(5)
25
{% endhighlight %}

Tanda noktah(.) dan kurungan(()) diperlukan.

## Alamak, Banyaknya Menaip!

Tidak pun, dan kita tidak suka mengeluh di sini.

Sebetulnya, Elixir ada mempunyai jalan pintas.(Di bawah menggunakan Elixir 0.9. Sintaksnya mungkin akan berubah di masa hadapan).  Kita terjun terus ke dalam kod.
{% highlight bash %}
iex> Enum.map [2,4,6], &1*&1
[4,16,36]
{% endhighlight %}

Apabila Elixir melihat simbol `&[1-9]`, ia tahu bahawa ia perlu menjana satu fungsi tanpa nama.  Fungsi tersebut akan mempunyai paramater dari `1` ke `n`, di mana `n` adalah nilai paling tinggi `&n`.  Badang fungsi itu pada asasnya ialah arahan mengandungi setiap ampersand(&), dengan setiap nilai ampersand dipetakan kepada parameter masing-masing.  Jadi, `&1*&1` secara logiknya adalah sama dengan `fn p1 -> p1*p1 end`, dan `rem(&1,&2)` menjadi `fn p1,p2 -> rem(p1,p2) end`.

Oleh sebab `fn` hanyalah nilai, anda boleh juga menulis seperti ini:
{% highlight bash %}
iex> square = &1 * &1
#Function<erl_eval.6.17052888>
iex> Enum.map [2,4,6], square
[4,16,36]
{% endhighlight %}

Ini adalah jalanpintas yang amat berguna, tetapi ada 'gotcha'. Apabila memutuskan kod untuk dimasukkan ke dalam badan fungsi fn, Elixir bermula dengan simbol ampersand dan melihat 'parse tree' untuk mendapatkan ekspresi penutup paling hampir.  Dengan `&1 * &1`, ianya operator darab.  Dengan rem(&1,&2), ianya panggilan kepada fungsi `rem`.

Tetapi anda akan mengalami kerumitan jika menulis `&1+&2+&3`.  Ini kerana, secara dalaman, ini digambarkan sebagai `(&1+&2)+&3`.  Elixir menterjemah ekspresi yang mengandungi `&1` dan `&2`, jadi sekarang kita mendapat `(fn p1,p2 -> p1+p2)+&3`.  Elixir akan komplen bahawa `&3` tidak dapat wujud tanpa kewujudan `&1` dan `&2`.  (Dan inilah mungkin kenapa sintaks ini akan diubahsuai).

Jadi, moralnya:  gunakan sintaks ampersand untuk ekspresi ringkas, dan `fn..end` untuk yang lain-lain.

## Fungsi Bernama

Fungsi Tanpa nama selalunya digunakan untuk operasi 'callback' - mereka selalunya pendek dan penggunaan mereka terhad.  Fungsi Bernama, adalah di mana kerja-kerja sebenar dijalankan.

Fungsi-fungsi Bernama cuma dapat wujud di dalam modul Elixir.  Sebaga contoh:
{% highlight ruby %}
defmodule AsciiDigit
  def valid?(character) do
  	character in ?0..?9
  end
end

IO.inspect AsciiDigit.valid? ?4 # => true
IO.inspect AsciiDigit.valid? ?a # => false
{% endhighlight %}

Untuk mengikuti kod ini, anda perlu ketahui bahawa sintaks ?x memulangkan kod aksara nombor bulat untuk x ( jadi ?0 ialah 48).

Contoh kita menetapkan satu modul bernama AsciiDigit mengandungi satu fungsi, valid?.  Ia mengambil satu kod aksara dan memulangkan `true` jika digit antara 0 hingga 9.  Kita gunakan operator `range`(..) untuk menetapkan aksara sah dari yang pertama hingga ke akhir, dan operator `in` untuk menguji 'inclusion'.

Elixir menyokong pemadanan corak apabila memastikan fungsi mana untuk dijalankan.  Anda boleh menggunakan `def` beberapa kali untuk fungsi yang sama, masing-masing dengan corak parameter yang berbeza.  Elixir akan memilih mana satu yang corak parameternya dapat dipadankan.

Teknik malar segar ini menunjukkannya dengan baik.  Spesifikasi urutan Fibonacci ialah
{% highlight bash %}
fib(0) = 1
fib(1) = 1
fib(n) = fib(n-2) + fib(n-1)
{% endhighlight %}

Ini bukan kod - ianya definasi matematik.  Sekarang, kita akan menukarkannya ke bentuk kod dengan mudah:
{% highlight ruby %}
defmodule Fibonucci do

	def fib(0), do: 1
	def fib(1), do: 1
	def fib(n), do: fib(n-2) + fib(n-1)

end

Enum.map 0..10, Fibonucci.fib(&1) #=> [1,1,2,3,5,8,13,21,34,55,89]
{% endhighlight %}
(saya tahu ini bukanlah satu cara efisien untuk memghitung urutan tersebut.  Saya mahu menunjukkan Elixir, dan tidak perlu bimbangkan prestasi algorithm).

Lain dari yang dapat dilihat, cuma ada satu tetapan untuk fungsi `fib`.  Ia cuma mempunyai tiga kepala - tiga corak argumen yang mempengaruhi pemilihan badan yang berbeza.

Dua kepala pertama memilih kes di mana argumen adalah `0` atau `1`.  Mereka menggunakan bentuk singkatan `do: expr` untuk memulangkan nilai `1`.  Langkah ketiga adalah langkah rekursif.  Jika tiada satu pun antara kedua-dua fungsi pertama padan dengan corak, fungsi ketiga akan dijalankan.

Apa berlaku jika kita menghantar argumen bernilai negatif.  Sekarang, ia akan membuat 'loop' sehingga kita kehabisan 'stack' atau sabar - menolak 1 atau 2 dari nilai negatif tidak akan mencapai 0.  Nasib baik, Elixir mempunyai klausa `guard`, yang membenarkan kita untuk meletakkan constraint tambahan pada pemadanan corak.
{% highlight ruby %}
defmodule Fibonacci do
 	
  def fib(0), do: 1
  def fib(1), do: 1
  def fib(n) when n > 1, do: fib(n-2)+fib(n-1)
 	
end
 	
Fibonacci.fib(10)   #=> 89
Fibonacci.fib(-10)
# => ** (FunctionClauseError) no function clause matching in Fibonacci.fib/1
{% endhighlight %}

Sekarang, jika kita memanggil `fib` dengan nombor negatif, Elixir tidak akan menjumpai fungsi klaus yang padan, jadi iakan menimbulkan ralat.  Jika anda mahu, anda boleh menguruskan kes ini di dalam kod, memberikan ralat yang lebih khusus aplikasi:
{% highlight ruby %}
defmodule Fibonacci do
 	
 	def fib(0), do: 1
 	def fib(1), do: 1
 	def fib(n) when is_integer(n) and  n > 1, do: fib(n-2)+fib(n-1)
 	def fib(x), do: raise "Can't find fib(#{x})"
 	
end
 	
Fibonacci.fib(10)     #=> 89
Fibonacci.fib(-10)    #=> ** (RuntimeError) Can't find fib(-10)
Fibonacci.fib("cat")  #=> ** (RuntimeError) Can't find fib(cat)
{% endhighlight %}


Kita menambah klaus `guard` untuk menguji jika parameter ialah nombor bulat, dan kemudian menambahkan satu fungsi kepala keempat yang menerima apa-apa parameter dan melaporkan ralat yang sepatutnya.


## Mengesahkan Nombor Kad Kredit

Kebanyakan rentetan nombor panjang yang kita uruskan setiap hari(nombor kad kredit, nombor IMEI di telefon dan lain-lain) mempunyai 'check digit'.  Ini selalunya angka terakhir nombor tersebut, dan dihitung menggunakan beberapa algorithm yang menjumlahkan semua digit-digit sebelumnya.  Jadi, apabila anda memasukkan nombor kad kredit, laman web akan menghitung semula 'check digit' dan mengesahkan ianya sama dengan digit terakhir nombor yang anda masukkan.  Ia bukan satu ujian untuk mengelakkan salahguna; ia cuma satu cara cepat untuk mengelakkan dari kesilapan menaip.

Mungkin teknik paling luas digunakan ialah `Luhn Algorithm`.  Ia menyongsangkan digit di dalam nombor, menceraikan mereka ke dalam dua set: digit di posisi ganjil dan digit di posisi genap.  Ia menjumlahkan angka-angka ganjil.  Untuk angka genap, ia mendarab setiap mereka dengan dua.  Jika keputusan ialah 10 atau lebih, ia akan menolak 9.  Kemudian ia akan menjumlahkan semua keputusan.  Menambahkan jumlah untuk posisi ganjil dan genap akan memberikan hasil yang boleh dibahagi 10 untuk nombor sah. 

Apabila saya bermula dengan Elixir, kepala saya masih penuh dengan cara konvensyenal untuk berbuat apa-apa.  Hasilnya saya menulis sesuatu seperti berikut:
{% highlight ruby %}
defmodule CheckDigit do
  import Enum

  def valid?(numbers) do
  	numbers = reverse(numbers)
  	numbers = map(numbers, fn char -> char - ?0 end)
  	numbers = map(numbers, fn digit, index -> {digit,index} end)
  	{ odds, evens } = partition(numbers, fn {_digit, index} -> rem(index,2) == 0 end)
  	sum_odd = reduce odds, 0, fn {number, _index}, sum -> sum +number end
  	sum_even = reduce evens, 0, fn {number, _index}, sum -> result = number * 2
  	    if result >= 10 do
  		    result = result -9
  	    end
  	    result + sum
    end
    rem(sum_odd + sum_even, 10) == 0
  end
end
{% endhighlight %}
Ugh! Mari kita langkah ke dalamnya.

Modul Enum mempuyai banyak fungsi untuk menguruskan 'collection'.  Kita akan menggunakan kebanyakan mereka, jadi kita akan import modul tersebut.  Ini bermakna kita akan menulis `map` dan bukannya `Enum.map`.

Fungsi `valid?` kita menerima satu list digit-digit UTF-8. 

Menggunakan keterangan `Luhn Algorithm`, kita songsangkan sususan digit, kemudian tukarkankan mereka dari digit UTF kepada nilai angka bulat sebenar( jadi `?1`, yang nilainya `41`, dipetakan ke `1`). Pada masa ini, jika diberikan argumen '123', kita akan mendapat list angka bulat [3,2,1].

Sekarang ia mula menjadi rumit.  Kita perlu untuk membahagikan digit-digit tersebut kepada mereka yang di posisi genap, dan mereka di posisi ganjil.  Sebagai persediaan, kita gunakan `map`, yang dihantar dengan fungsi `fn number, index -> {number,index} end`.  Fungsi ini mengambil nilai angka bulat di dalam list tersebut, dan petakan kepada satu tuple yang mengandungi setiap satu.

Pada titik ini, lampu amaran sepatutnya kedengaran.  Ini terlalu sukar.  Tapi kita teruskan juga, kerana ini apa yang pengaturcara lakukan.

Fungsi `partition` mengambil satu 'collection' dan satu fungsi.  Ia memulangkan satu tuple di mana elemen pertama ialah satu list yang mengandungi nilai-nilai yang dipulangkan sebagai `true`, dan elemen kedua adalah nilai-nilai lain.

Sekarang kita perlu menjumlahkan nilai-nilai ganjil.  Setiap kali anda perlu untuk me-reduce satu 'collection' kepada satu nilai, anda mungkin perlu gunakan fungsi `reduce`.  Ia mengambil 'collection' tersebut, satu nilai asas, dan satu fungsi.  Fungsi ini menerima setiap elemen di dalam 'collection' tersebut berturutan, bersama dengan nilai semasa.  Apa-apa yang dipulangkan oleh fungsi tersebut menjadi nilai semasa seterusnya.  Jadi, menjumlahkan satu senarai nombor boleh dilakukan dengan:
{% highlight ruby %}
Enum.reduce list, fn val, sum -> val + sum end
# atau
Enum.reduce list, &1 + &2
{% endhighlight %}

Tapi apa yang kita ada adalah satu senarai tuple {value,index}.  Ini bermakna kita perlu menggunakan pemadanan corak keatas parameter pertama fungsi tersebut untuk mengekstrak nilai.( Simbol `_` di hadapan `_index` bermaksud kita tidak akan mempedulikannya).


Menjumlahkan nombor genap menggunakan kaedah yang sama, tetapi kita perlu membuat penggandaan, dan penukaran nombor sepuluh ke atas.


Di akhir semua ini, kita boleh menguji melalui `iex`, seperti berikut:
{% highlight bash %}
$ iex validate_cc.ex
iex> CheckDigit.valid? '4012888888881881'
true
iex> CheckDigit.valid? '0412888888881881'
false
{% endhighlight %}


## Pengemasan

Penyelesaian kita berjaya, tetapi gayanya adalah tidak berbentuk kefungsian.(Ianya cara sopan untuk mengatakan bahawa ia terlalu hodoh).  Untuk mengemaskannya,  saya mencari tempat-tempat yang jelas ada kesilapan, dan melihat jika boleh saya betulkan.


Masalah pertama yang saya nampak ialah tiga baris pertama.  Saya menukarkan('transform') nombor-nombor yang diberikan kepada satu senarai digit dalam susunan songsang, setiap satu mempunyai indeks.

Perkataan penukaran('transform') itu adalah kata kuncinya.  Aturcara Kefungsian adalah mengenai penukaran data('transforming data').  Ianya cukup penting sehingga Elixir mempunyai satu operator, `\|>`.  Ia membenarkan kita untuk membina satu sambungan fumgsi-fungsi, di mana setiap satu akan men-transformasi hasil dari fungsi sebelumnya.  Ia membenarkan kita untuk mengarang kefungsian.

Menggunakan operator 'pipeline', kita boleh menulis semula tiga baris pertama tersebut sebagai:
{% highlight ruby %}
numbers
  |> reverse
  |> map(fn char -> char -?0 end)
  |> map(fn digit, index -> { digit, index } end)
{% endhighlight %}


Kita ambil list asal, membuat transformasi dengan menyongsangkan urutannya, kemudian menukarkan kod aksara kepada  nombor bulat, dan kemudian sekali menambahkan indeks untuk elemen tersebut.

Operator 'pipeline' nampaks seperti magik, tetapi sebenarnya agak mudah.  Ia mengambil nilai ekspresi di sebelah kiri, dan memasukkannya sebagai argumen pertama untuk fungsi di kanannya, dan seterusnya membuat pemindahan sebegitu terus ke bawah.


Isu kedua yang akan kita lihat ialah berkenaan dengan pembahagian dan penjumlahan.  Masalahnya ialah kita berfikir secara imperatif, dan bukan secara kefungsian.  Kita memberitahu ELixir setiap langkah yang perlu dibuat, padahal sepatutnya kita berfikir tentang spesifikasi apa yang mahu kita buat, dan membiarkannya menghasilkan keperincian.

Fikir balik kepada contoh Fibonacci kita.  Di situ kita mengimplementasikan spefikasi kita sebagai tiga kepala fungsi, yang memadankan dua kes khusus dan satu kes am.  Bolehkah kita lakukan sedemikian di sini?


Daripada kita memproses list digit kita satu elemen pada satu masa, apa kat kita memproses dua digit serentak?  Ini bermakna kita bekerja dengan sepasang digit - satu yang berada di posisi genap, dan lagi satu dari posisi ganjil.  Kita tahu cara untuk menjumlahkan nombor Luhn untuk pasangan digit tersebut, dan kemudian kita akan tambahkan jumlah tersebut kepada jumlah dari baki list tersebut.  Itulah langkah rekursif kita.


Apabila kita akhirnya mengosongkan list tersebut, kita sudahpun membuat penjumlahan yang diperlukan, jadi kita cuma perlu memulangkan nilai itu sahaja.


Terdapat lagi satu kes untuk dibincangkan. Jika list tersebut mempunyai bilangan digit ganjil, jadi apabila kita sampai ke akhir, kita akan mempunyai hanya satu elemen.  Tetapi kita tahu elemen tersebut berada pada posisi gnajil, jadi kita hanya perlu menambahkan nilai tersebut kepada jumlah sedia ada dan pulangkan hasilnya.

Jadi, berikut adalah versi baru kod kita:
{% highlight ruby %}
defmodule CheckDigit do
  import Enum

  @doc """
  Pastikan jika satu urutan digit adalah sah, dengan menganggap digit terakhir adalah sejenis 'Luhn checksum'(http://en.widipedia.org/wiki/Luhn_algorithm)
  """
  def valid?(numbers) when is_list(numbers) do
  	numbers
  		|> reverse
  		|> map(&1 - ?0)
  		|> sum
  		|> rem(10) == 0
  end

  defp sum(digits), do: _sum(digits, 0)
  defp _sum([], sum), do: sum
  defp _sum([odd], sum), do: sum + odd
  defp_sum([odd, even | tail], sum) when even < 5 do
  	_sum(tail, sum + odd + even*2)
  end
  defp _sum([odd, even | tail], sum) do
  	_sum(tail, sum + odd + even*2 - 9)
  end
end
{% endhighlight %}

'Pipeline' di bahagian atas adalah lebih ringkas - tidak lagi perlu menguruskan indeks -indeks, tiada lagi pembolehubah sementara.  Ia dibaca seperti versi kod kepada spesifikasi.

Fungsi `sum` adalah contoh kepada corak lazim.  Kita perlu menetapkan nilai awal kepada nilai yang akan dijumlahka, tetapi kita tidak mahu kod yang memanggil kod ini mnegetahui keperinciannya, jadi kita menulis satu versi fungsi `sum` yang hanya mengambil nombor-nombor tersebut, kemudian memanggil fungsi yang melaksanakan implementasi sebenar, hantarkan satu list dan nilai awalan sifar.  Kita boleh gunakan nama yang sama untuk fungsi-fungsi sokongan tersebut, tetapi saya lebih suka menggunakan `_sum` untuk membezakan mereka.(Ramai pengaturcara Elixir menamakan mereka sebagai `do_sum`, tetapi bagi saya ianya nampak terlalu imperatif.)

Fungsi-fungsi `_sum` mempunyai 4 kepala:

- Jika list kosong, kita pulangkan jumlah yang telah kita kumpulkan sebagai parameter kedua.
- Jika list mengandungi satu elemen, tambahkan nilai elemen tersebut kepada jumlah yang sedang dikumpulkan dan pulangkan nilai tersebut.  Ii akan mematikan proses apabila dalam keadaan di mana list tersebut mengandungi bilangan digit ganjil.
-  Jika tidak, kita ekstrak dua elemen pertama dari dalam list.  Ini menggunakan corak `[odd, even | tail]`.  Elemen pertama dikaitkan dengan `odd`, yang kedua dengan `even`, dan baki list dikaitkan dengan `tail`.  Melihat balik kepada algorithm Luhn, kita mempunyai dua kes untuk diuruskan.  Jika hasil dari mendarab nombor posisi genap dengan dua adalah kurang dari sepuluh, nilai itulah yang akan kita tambah kepada jumlah.  Kita gunakan klausa `guard` untuk memeriksa keadaan ini.
-  Jika selainnya, kita akan menolak sembilan dari hasil darab tersebut.  Ini adalah apa yang badan fungsi keempat lakukan.

Perhatikan bagaimana kita menghantar jumlah yang dikemaskini sebagai paramater kedua fungsi-fungsi tersebut - ini adalah corak unversal apabila anda mahu mengumpulkan nilai dari satu set panggilan fungsi-fungsi rekursif.

## Apa Yang Lain Mengenai Kod Ini

Apabila kita menulis di dalam bahasa seperti Java, C# atau Ruby, kita bekerja di beberapa peringkat.  Sebahagian dari otak kita berfikir mengenai spesifikasi - apa yang perlu dilakukan. Sebahagian lagi berfikir mengenai perlaksanaan - bagaimana ia perlu dilakukan.  Dan di situlah kita selalu tersekat.

Lihat contoh yang terakhir.  Kita membuat iterasi ke atas satu set digit.  Kita memilih mereka yang berada di posisi genap dan posisi ganjil.  Kita melakukan pengiraan conditional.  Dan tidak ada satu pun struktur kawalan di dalam program tersebut.  Tiada `if`, tiada `loop`.  Kod yang ditulis adalah lebih kepada gambaran kepada spesifikasi apa yang kita mahukan.

Dan itulah salah satu sebab saya menjadi seorang peminat aturcara kefungsian secara am, dan khususnya Elixir.

>Dave Thomas ialah seorang pengaturcara yang suka menyebarkan perkara yang hebat.  Beliau adalah salah seorang penulis buku Pragmatic Programmer, dan salah seorang pencipta Agile Manifesto.  Buku beliau Programming Ruby memperkenalkan bahasa Ruby kepada dunia, Agile Web Development with Rails membantu memulakan revolusi Rails. 




