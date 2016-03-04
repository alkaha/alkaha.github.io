---
layout: post
title: Pattern Matching melalui Elixir
---

- Dokumen asal: [https://quickleft.com/blog/pattern-matching-elixir/](https://quickleft.com/blog/pattern-matching-elixir/)
- Penulis asal: [Trace Helms](https://quickleft.com/blog/author/thelms/)
- Ringkasan: Dokumen ini memperkenalkan penggunaan 'pattern matching' melalui Elixir. Ia juga memberikan pengenalan awal kepada Elixir dan Functional Programming.  Dokumen ini menganggap anda mempunyai penegenalan asas mengenai Elixir.

---

### Pengenalan

Mempelajari 'programming language' baharu adalah satu cara yang baik untuk mengembangkan pengetahuan kita mengenai programming.  Dengannya kita akan dapat menambah teknik-teknik yang baharu dan juga memberikan pandangan dan cara berfikir yang baharu. Salah satu perkara yang dapat saya cungkil dari Elixir adalah 'pattern matching'.  Ianya timbul dari penggunaan di dalam Erlang dan bagi saya ianya salah satu elemen yang saya harap ada pada kebanyakan 'programming language'.

### Apa itu 'Pattern Matching'?

Di dalam Elixir, simbol `=` tidak semestinya bermaksud "ubah nilai satu variable kepada nilai yang lain".  Sebaliknya ia bermakna "samakan bahagian kiri dengan bahagian kanan".  Biasanya kita akan menulis `x = 2` dan tidak nampak apa yag berbeza.  Tetapi apabila sampai satu peringkat, kita akan nampak sesuatu yang agak berbeza.

{% highlight js %}
iex(1)> x = 2
2
iex(2)> y = 3
3
iex(3)> 2 = x
2 # boleh begini?
iex(4)> 2 = y
** (MatchError) no match of right hand side value: 3
{% endhighlight %}

Ini tidak bermaksud nombor `2` diberikan nilai `x` tetapi ianya di-'pattern match' dengan `x`.  Perbezaannya agak tidak ketara buat masa ini, jadi mari kita lihat bagaimana ianya berguna.

### Nota Mengenai 'Immutable Variables (variable yang nilainya tidak boleh diubah)'

Elixir ialah sejenis 'functional language' yang bermakna nilai sesatu 'variable' tidak boleh diubah.  Walaupun begitu, kita boleh mengulang-guna 'variable' tersebut.  Sekiranya kita menulis `x = 2`, dan kemudian `x = 3`, ianya dibenarkan oleh Elixir.

{% highlight js%} 
iex> x = 2
2
iex> y = 3
3
iex> x = y
3
iex> x
3
iex> ^x = 2 # force no reusing of variables
** (MatchError) no match of right hand side value: 2
{% endhighlight %}

Walaupun di pengakhir kod di atas, kita telah `x` telah disesuaikan dengan nilai `3`, tetapi dibelakang tabir, masih lagi terdapat variable `x` yang disesuaikan dengan nilai asal, iaitu `2`.  Sekiranya anda menghantar 'variable' `x` kepada satu proses yang sedang berjalan secara selari, proses tersebut akan menerima 'variable' `x` yang mempunyai nilai `2`.  Kod di atas menunjukkan kita mengulangguna 'variable' `x` dengan memberikannya nilai berbeza.  Kita juga boleh menghalangnya dari berlaku dengan menggunakan simbol `^`, contohnya `^x`.  Contohnya kod di atas menunjukkan ralat apabila kita menaip `^x = 2`, kerana buat masa ini `x` sedang disuaikan dengan `3`.  Tetapi jika menaip `^x = 3`, ianya dibenarkan kerana nilai semasa `x` adalah `3`.  

### Asas 'Pattern Matching'

Jika kita ada satu 'tuple' dan kita mahu meletakkan nilai isi kandungan 'tuple' tersebut kepada beberapa 'variable' yang berbeza sebagaimana berikut:

{% highlight js%} 
iex> {result, value} = {:ok, 2}
{:ok, 2}
iex> result
:ok
iex> value
2
{% endhighlight %}

Ini menunjukkan kita dapat mengambil nilai di dalam tuple tersebut kepada variable yang berlainan.
Tapi jika kita ingin memberikan nilai kepada variable `value` hanya jika `result` adalah `:ok`?

{% highlight js%} 
iex> {:ok, value} = {:ok, 2}
{:ok, 2} # 'pattern match' ini berjaya
iex> value
2
iex> {:ok, value} = {:nope, 2}
** (MatchError) no match of right hand side value: {:nope, 2}
{% endhighlight %}

Ini sebenarnya menyesuaikan `:ok` dari sebelah kiri dengan yang di sebelah kanan. `value` diberikan ilai `2` hanya kerana keseluruhan nilai tuple di sebelah kiri dapat disesuaikan dengan tuple di sebelah kanan. Penyesuaian kedua, iaitu `{:ok, value} = {:nope, 2}` tidak berjaya kerana `:ok` tidak sama dengan `:nope`.  Lihat bagaimana kita boleh menggunakan konsep ini di dalam kod berikut:

{% highlight js%} 
# my_case.exs
defmodule MyCase do

  def do_something(tuple) do
    case tuple do
      {:ok, value} ->
        "The status was :ok!"
      {:nope, value} ->
        "Nope nope nope nope..."
      _ ->
        "You passed in something else."
    end
  end

end
{% endhighlight %}

Kemudian jalankan fail ini melalui `iex` dengan menaip `$ iex my_case.exs`.

{% highlight js%} 
iex> MyCase.do_something({:ok, true})
"The status was :ok!"
iex> MyCase.do_something({:nope, true})
"Nope nope nope nope..."
iex> MyCase.do_something({:wat, true})
"You passed in something else."
{% endhighlight %}

Saya berharap ianya menjelaskan faedah penggunaan 'pattern matching' di dalam Elixir.  Sekiranya digunakan dengan betul, ia akan membuat kod yang kita tulis senang untuk dibaca.

### Penggunaan 'Pattern Matching' Ketika Membuat Function

Ini adalah situasi di mana 'pattern matching' paling berguna dan paling kerap digunakan di dalam Elixir.  Kita akan menulis 'function' yang hanya akan dijalankan apabila 'argument' 'function' tersebut adalah sama dengan 'pattern' yang diuji. Sebagai contoh:
{% highlight js%} 
# talker.exs
defmodule Talker do

  def say_hello(:bob), do: "Hello, Bob!"
  def say_hello(:jane), do: "Hi there, Jane!"
  def say_hello(name) do # name is a variable that gets assigned
    "Whatever, #{name}."
  end

end
{% endhighlight %}

Jalankan fail ini dengan menaip `$ iex talker.exs`
{% highlight js%} 
iex(6)> Talker.say_hello(:bob)           
"Hello, Bob!"
iex(7)> Talker.say_hello(:jane)          
"Hi there, Jane!"
iex(8)> Talker.say_hello('random person')
"Whatever, random person."
{% endhighlight %}

Kita dapat lihat ketiga-tiga 'function' tersebut (yang diberikan nama yang sama) dijalankan mengikut 'argument' yang dibekalkan.  Ini memberikan asas untuk kepada beberapa perkara yang hebat, seperti 'recursion' yang begitu banyak menggunakan 'pattern matching', yang akan kita lihat seterusnya.

### Kegunaan 'Pattern Matching' di dalam 'Recursion'

Kita akan menulis satu 'function' yang akan menukar nilai setiap nombor di dalam satu 'list' kepada ganda kuasa 2 (contohnya menukar [2,3,4] kepada [4,9,16]). Kita mulakan dengan menulis 'function' yang menghasilkan 'list' kosong, iaitu `[]`.  Kemudian kita menulis 'function' yang membuat operasi arithmetik yang diperlukan. 

{% highlight js%} 
# my_squarer.exs
defmodule MySquarer do

  def square([]), do: []
  def square([head | tail]) do
    [head * head | square(tail) ]
  end

end
{% endhighlight %}

Di dalam Elixir, kita boleh menyatakan satu 'list' sebagai elemen pertama('head') dan satu senarai elemen yang seterusnya. Elemen terakhir di dalam satu 'list' adalah satu senarai kosong([]).  Jadi apabila kita membuat kenyataan `[head|tail]`, kita sebenarnya membuat penyesuaian element pertama kepada `head` dan elemen yang selebihnya kepada `tail`.

{% highlight js%} 
# contoh menjelaskan apa itu 'head' dan 'tail'
iex> [head | tail] = [1, 2, 3, 4]
[1, 2, 3, 4]
iex> head
1
iex> tail
[2, 3, 4]
iex> [head | tail] = [1]
[1]
iex> head
1
iex> tail
[] 
{% endhighlight %}

Berbalik kepada 'function' `square\1` kita. Ia akan menghasilkan satu 'list' baru yang mengandungi ganda kuasa dua elemen pertama di dalam 'list' yang digunakan sebagai 'argument', kemudiannya mengulangpanggil `square\1` terhadap elemen selebihnya di dalam 'list' tersebut.  Apabila sampai kepada elemen terakhir di dalam 'list' tersebut, iaitu 'list' kosong([]), ia akan menjalankan 'function' `square([])` yang akan menghasilkan `[]` dan seterusnya memberhentikan 'recursion' tersebut.

Untuk melihat hasilnya, jalankan fail tersebut dengan menaip `$ iex my_squarer.exs`.

{% highlight js%} 
iex> MySquarer.square([])
[]
iex> MySquarer.square([1, 2, 3])
[1, 4, 9]
iex> MySquarer.square([-1, -2, -3])
[1, 4, 9]
{% endhighlight %}

### Penutup
Saya berharap sekarang anda dapat lebih memahami apa itu 'pattern matching' di dalam Elixir dan bagaimana ianya bermanafaat. Teknik 'recursion' memang agak sukar untuk difahami, tetapi 'pattern matching' di dalam Elixir memberikan peluang kepada kita untuk lebih mudah untuk menulis dan memahaminya. 

