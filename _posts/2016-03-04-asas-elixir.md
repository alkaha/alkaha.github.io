---
layout: post
title: Asas Elixir
---

- Dokumen asal: [https://quickleft.com/blog/elementary-elixir/](https://quickleft.com/blog/elementary-elixir/)
- Penulis asal: [Trace Helms](https://quickleft.com/blog/author/thelms/)
- Ringkasan: Dokumen ini memperkenalkan perkara-perkara asas yang perlu diketahui mengenai Elixir.

---

Di dalam post yang lepas, saya telah memperkenalkan ['pattern-matching' melalui Elixir](http://alkaha.github.io/2016/03/03/pattern-matching-melalui-elixir/), satu bahasa aturcara baharu yang dibina oleh ahli teras Rails, iaitu José Valim.  [Elixir](http://elixir-lang.org/) dijalankan di atas [Erlang](http://www.erlang.org/) dan baru-baru ini telah melancarkan v1.2. Terdapat banyak kelebihan yang ada pada Elixir, termasuk kelajuan tinggi, konkurensi9'concurrency'), tahan rosak('fault-tolerance') dan lain-lain.

Di dalam artikel ini saya akan menunjukkan beberapa kelebihan asas Elixir.  Untuk melihat contoh kegunaan sebenar penggunaan bahasa ini, anda boleh menonton video ini: [Engineering Lunch](http://go.quickleft.com/elementary-elixir).  Anda juga boleh mendapatkan slaid-slaid saya di laman tersebut.

## Pengaturcaraan Kefungsian(Functional Programming)

Dengan Elixir, proses-proses tidak berkongsi ruang ingatan('memory').  Ini bermakna fungsi yang sama di dalam proses yang berbeza tidak boleh menukar nilai pembolehubah yang telah diberikan oleh fungsi di dalam proses yang lain dan proses 'garbage collection' tidak akan memperlahankan atau menghalang aplikasi anda dari berfungsi.  Sebagai sejenis bahasa kefungsian('functional language'), ianya memberikan banyak kemudahan kerana kita tidak perlu bimbang mengenai ingatan kongsi('shared memory') dan keadaan kongsi('shared state').  Ianya sejenis aturcara yang agak berbeza, tetapi jika kita dapat mengatasi halangan-halangan awal, ianya memberikan banyak kemudahan dan kebaikan yang diberikan oleh Elixir.

## Konkurensi (menjalankan proses secara selari)

Proses-proses kecil dan ringan menggunakan banyak 'core' dan mesin.  Elixir menghasilkan banyak proses-proses kecil yang menggunakan ruang ingatan mereka sendiri dan berkomunikasi melalui pemindahan mesej antara satu sama lain.  

Ini bukanlah proses-proses 'operating system' yang besar dan berat.  Ini adalah proses-proses kecil Elixir dan mereka adalah sangat kecil dan ringan.  Di dalam video di atas, kami menghasilkan 1 juta proses-proses ini dalam masa cuma 8.3 saat.  Proses-proses ini juga menggunakan kesemua 'core' di dalam mesin, yang menyebabkan operasi secara selari amat efisien di dalam aplikasi anda.  Tiada lagi menghantar proses-proses kepada 'background job worker'.  Di dalam Elixir, 'background worker' anda telah siap dipasang.

### Demonstrasi

Chris McCord, pencipta [Phoenix, satu web framework untuk Elixir](http://www.phoenixframework.org/), telah [menguji tahap kecekapan Elixir](https://twitter.com/chris_mccord/status/659430661942550528) dengan melihat jumlah hubungan konkurensi yang dapat dicapai menggunakan hanya satu pelayan.  Cara pengujian beliau adalah dengan membina satu 'chat room' untuk melihat bagaimana mereka dapat meghantar satu pesanan kepada semua pengguna di dalam 'chat room' tersebut.  Ia menggunakan pelayan yang agak besar, iaitu 40 core, tetapi Phoenix dan Elixir telah berjaya menggunakan kesemua 'core' di dalam pelayan tersebut dan menghasilkan prestasi yang amat baik.

2 juta pengguna maya telah diletakkan di dalam 'chat room' tersebut dan Phoenix berjaya menghantar pesanan tersebut kepada kesemua pengguna.  Ia cuma mengambil masa satu atau dua saat untuk setiap pengguna menerima pesanan tersebut, kesemua dua juta pengguna!  Ianya amat menakjubkan dan merupakan satu demonstrasi yang baik untuk menunjukkan kebolehan 'framework' tersebut.

## Tahan Rosak(Fault Tolerance)

Anda boleh membuat pengawasan terhadap proses-proses anda dengan menggunakan 'Supervisor'.  Komponen Elixir ini (sebenarnya dari OTP Erlang), memberikan kemudahan untuk anda membentuk strategi untuk menghidupkan, memberhetikan dan menghidupkan semula proses-proses yang berhenti secara mengejut.  Sebagai contoh, sekiranya salah satu dari proses anda berhenti secara mengejut, Elixir akan menghidupkan semula proses tersebut tanpa campurtangan anda.

## Komponen Bahasa

### Pemadanan Corak(Pattern Matching)

Elixir menggunakan kaedah pemadanan corak('pattern matching') dan bukannya memberikan nilai kepada pembolehubah.  Ianya merupakan satu cara berfikir yang agak berbeza, tetapi membenarkan beberapa kaedah programming yang agak berkuasa.  Jika anda tidak begitu jelas mengenai pemadanan corak('pattern matching') melalui Elixir, saya cadangkan melihat artikel saya mengenai pemadanan corak('pattern matching'): [Pemadanan Corak(Pattern Matching) Melalui Elixir](http://alkaha.github.io/2016/03/03/pattern-matching-melalui-elixir).

### Aksara Paip(Pipe Character, |> )

Aksara paip digunakan untuk membersihkan kod yang ditulis dan menyebabkan tujuan kod itu ditulis dapat difahami dengan lebih jelas.  Lebih kurang sama dengan aksara paip di dalam Unix, ianya adalah cara untuk mengatakan "ambilkan nilai yang dihasilkan oleh fungsi ini dan hantarkannya kepada fungsi yang berikutnya".  Daripada menulis kod seperti ini:
{% highlight bash %}
process(parse_arg(args))
{% endhighlight %}
kita akan menulis seperti ini:
{% highlight bash %}
args |> pass_arg |> process
{% endhighlight %}
Perbezaannya agak tidak ketara, tetapi akan memberikan impak yang besar apabila menulis kod untuk operasi yang lebih besar.

### Rekusi(Recursion)

####  List

Kita mulakan dengan 'list'.  Setiap 'list' di dalam Elixir adalah di dalam bentuk 'linked list' dan boleh diceraikan kepada bahagian elemen yang pertama dan bahagian elemen-elemen yang seterusnya.  Dengan menggunakan pemadanan corak('pattern matching'), kita boleh memberikan nilai elemen-elemen ini kepada pembolehubah dengan mudah:
{% highlight bash %}
iex> [head | tail] = [1, 2, 3, 4] 
[1, 2, 3, 4] 
iex> head 
1 
iex> tail 
[2, 3, 4]
{% endhighlight %} 

Elemen terakhir di dalam 'list' Elixir adalah satu 'list' kosong, sebagaimana berikut:
{% highlight bash %}
iex> [head | tail] = [1] 
[1] 
iex> head 
1 
iex> tail 
[]
{% endhighlight %} 

#### Fungsi Rekursif(Recursive Functions)

Disebabkan oleh peggunaan pemadanan corak('pattern matching') di dalam takrifan fungsi('function definition'), kita dibolehkan untuk menulis fungsi rekursif('recursive function') sebagai beberapa fungsi.  Fungsi pertama adalah sebagai fungsi tapak('base function'), dan fungsi-fungsi seterusnya adalah untuk menjalankan operasi-operasi seterusnya:
{% highlight ruby %}
defmodule MyList do 

  def square([]), do: [] 

  def square([head | tail]) do 
    [head * head | square(tail) ] 
  end

end
{% endhighlight %} 

Jadi apabila kita memanggil `MyList.square([])`, kita menggunakan fungsi yang pertama dan akan menghasilkan `[]`.  Apabila kita memanggil `MyList.square([1])`, fungsi pertama tidak dapat memenuhi 'pattern []', jadi ia tidak digunakan dan kita akan gunakan fungsi yang kedua.  Fungsi kedua akan menyesuaikan `head = 1` dan `tail = []`.  Ini akan menghasilkan `[1 * 1] | square([])`.  Ini akan mengulangpanggil fungsi `square()` dan membekalkan `[]` sebagai 'argument', yang seterusnya akan menghentikan proses tersebut.

Perkara yang sama berlaku apabila kita memanggil `MyList.square([1,2,3,4])`, yang akan menghasilkan `[1,4,9,16,[]]`, ataupun sebenarnya [1,4,9,16]`.

Teknik-teknik sebegini menjadi asas kepada rekursi('recursion') di dalam Elixir.

#### Enum Library

Enum Library di dalam Elixir menggunakan kaedah rekursi dan ianya amat hebat.  Ini bermakna kita tidak perlu menulis fungsi rekursif kita sendiri untuk kebanyakan operasi yang melibatkan 'list':
{% highlight bash %}
iex> Enum.sum([1, 2, 3, 4])
10
{% endhighlight %}

## Microservices 

Microservices adalah hangat diperkatakan masa ini.  Ia membenarkan kita memasang dan meningkat skala sebahagian dari aplikasi kita secara berasingan dan menyimpan asas kod kita berasingan.  Dengan Elixir, kita mempunyai konsep 'Applications'.  'Applications' membenarkan kita membuat 'package' kepada sebahagian dari kod aplikasi kita dan menjalankan mereka di bawah satu 'supervision tree'.  Pada asasnya mereka boleh dipanggil sebagai 'microservices'.  Kita boleh membuat konfigurasi bagaimana kita mahu proses-proses kecil ini dihidupkan, dimatikan dan diawasi.  Paling penting, proses-proses ini berjalan secara selari dan tidak berkogsi ruang 'memory' atau 'state'.  Jadi kita mendapat kelebihan penggunaan 'microservices' dan menguruskan proses-proses tersebut dari satu tempat yang sama.

## Pengagihan(Distribution)

Dengan penggunaan OTP, Elixir didatangkan dengan cara mudah untuk membuat pengagihan kod.  Perlu untuk menambah saiz secara melintang ('scale horizontally')? Kita cuma perlu menghidupkan beberapa 'server' baru, sambungkan kepada kod kita, dan kita terus boleh beroperasi. Ianya akan menjadi agak sukar untuk dilaksanakan, tetapi secara asasnya, ia menggunakan proses yang sama.  Saya menunjukkan contoh penggunaan konsep ini di dalam video 'Engineering Lunch' di atas.  Semasa persembahan itu, saya menyambungkan 'laptop' rakan sekerja saya melalui wi-fi dan menjalankan kod saya melalui 'laptop' beliau. Ianya kod yang sengaja dibuat untuk berjalan secara perlahan dan kita boleh melihat bagaimana 'processor' mesin beliau bekerja secara maksima kepada 100% untuk cuba menjalankan kod tersebut.  Saya anggarkan yang proses pengagihan kod ini akan megambil masa yang lama, tetapi kami telah berjaya melakukannya dalam masa 5 minit.

Pastikan anda menonton video [Engineering Lunch](http://go.quickleft.com/elementary-elixir) untuk melihat bagaimana ia berfungsi.     


