---
layout: post
title: Belajar Elixir Dengan Membina Pencari Happy Number
---

- Dokumen Asal: [Learning Elixir by Building a Happy Number Finder](https://medium.com/the-many/learning-elixir-by-building-a-happy-number-finder-652b0b4b0f2d#.i6bcw2d15)
- Penulis Asal: [Scott Luptowski](https://medium.com/@scottluptowski)
- Tarikh Artikel Asal: Feb 11, 2016

---

Baru-baru ini saya mula belajar Elixir dan memutuskan untuk membina satu aplikasi Pencari Happy Number.  Ini adalah salah satu masalah aturcara kegemaran saya - tidak seperti namanya, ia tidak melibatkan banyak matematik.  Sebaliknya, penyelesaian kepada Pencari Happy Number melibatkan rekursi, penukaran type, dan pemahaman list, semua perkara penting untuk diketahui apabila meneroka satu bahasa baru.

### Apa itu Happy Number?

Mari tanya Wikipedia: "Bermula dengan mana-mana angka bulat positif, tukarkan nombor itu dengan hasil tambah kuasa dua digit-digit nombor itu, dan ulangkan proses tersebut sehingga nombor itu bersamaan dengan 1[happy number], atau ia bergelung tanpa henti di dalam kitaran yang tidak mengandungi 1[unhappy number]"

---

### Mari bermula

Kita akan menunjukkan proses di atas dengan mencari happy number untuk 49.  Pertama sekali kita perlu mencari jalan untuk menceraikan nombor 49 kepada digit 4 dan digit 9.  Kemudian kita perlu kuasa dua kan nombor-nombor tersebut dan jumlahkan hasil-hasilnya.  Jika nombor itu ialah 1, ianya satu happy number dan kita telah siap.  Jika nombor itu bukan 1, kita menguji semula dengan nombor lain dan lihat jika mendapat nombor 1.

Dalam banyak bahasa lain, termasuk Javascript dan Ruby, menceraikan 49 kepada 4 dan 9 memerlukan anda menukarkan nombor kepada string supaya anda boleh menceraikan string tersebut kepada aksara-aksara berasingan dan kemudian menukarkan setiap aksara tersebut balik kepada nombor bulat.  Tidak dengan Elixir - anda boleh hantar apa-apa nombor ke dalam fungsi `Integer.digits/2` untuk mendapatkan satu list digit-digit di dalamnya.

> Nota: Di dalam Elixir, fungsi-fungsi dirujuk melalui nama mereka dan berapa argumen yang diharapkan, sebagai contoh `digits/2`.  Di dalam kes fungsi ini, argumen keduanya adalah tidak wajib.

Seterusnya, kita mesti gandakan setiap nombor di dalam list dan jumlahkan list tersebut.  Kita akan buat dulu menggunakan jalan panjang sebelum membina penyelesaian ala-Elixir.

```elixir
defmodule HappyNumberFinder do
  def get_sum(number) do
    numbers = Integer.digits(number) #=> [4, 9]
    squared_numbers = Enum.map(numbers, fn(x) -> x * x end ) #=>[16, 81]
    sum = Enum.reduce(squared_numbers, fn(x, total) -> x + total end)
  end
end
```

Hasilnya ialah 97, yang mana bukan 1.  Kita tahu kita perlu memanggil `get_sum/1` dengan 97.  Tetapi sebelum kita memperkenalkan rekursi kepada program ini kita patut membuat sedikit refaktor untuk mendapat penyelesaian yang lebih idiomatik Elixir.  Kita boleh gunakan operator paip `'|'>` untuk mempaipkan hasil dari fungsi sebelum kepada fungsi seterusnya.

```elixir
defmodule HappyNumberFinder do
  def get_sum(number) do
    number 
    |> Integer.digits #=> [4, 9]
    |> Enum.map(fn(x) -> x * x end ) #=>[16, 81]
    |> Enum.reduce(fn(x, total) -> x + total end) #=> 97
  end
end
```

Kita boleh bersihkan sedikit kod ini dan gunakan operator `&`. Jalanpintas ini membenarkan anda menukarkan satu ekspresi kepada satu fungsi membungkus panggilan itu dengan `&` dan menamakan paramater-parameter anda `&1,&2` dan lain-lain: sebagai contoh, `(&1 + 1)` adalah sama dengan `fn(x) -> x + 1 end`.

```elixir
defmodule HappyNumberFinder do
  def get_sum(number) do
    number 
    |> Integer.digits #=> [4, 9]
    |> Enum.map(&(&1 * &1) ) #=>[16, 81]
    |> Enum.reduce(&(&1 + &2)) #=> 97
  end
end
```

Terdapat peringkat di mana kerapian menghindarkan kebolehbacaan keseluruhan dan bagi saya contoh paip pertama adalah lebih jelas tujuannya daripada contoh kedua.  Walaupun begitu, kita teruskan.

Sekarang kita memiliki satu fungsi yang menentukan sesatu nombor itu adalah happy number.  Mari kita sediakan rekursi tersebut.

```elixir
defmodule HappyNumberFinder do

  # def get_sum(number)

  def find(number) do
    sum = get_sum(number)

    case sum do
      1 -> {:happy}
      _ -> find(sum)
    end
  end

end
```

Kita dapat lihat kod ini beraksi dengan nombor 49, yang mana merupakan satu happy number.

```elixir
HappyNumberFinder.find(49) #=> {:happy}
```

Hebat!  Bagaimana pula jika kita panggilkan dengan nombor bukan happy number?

```elixir
HappyNumberFinder.find(400) #=> ???
```

Kita akan dapati tetingkap terminal kita beku.  Anda tidak sepatutya nampak sebarang paparan pada tetingkap terminal, tetapi Elixir sedang bekerja keras - kita sedang membuat rekursi tanpa henti.  Ianya berguna jika kita mengetahui nilai apa yang sedang diproses oleh Elixir.  Kita akan melanggar undang-undang Elixir buat sementara waktu dengan memperkenalkan satu kesan sampingan kepada fungsi kita, dalam kes ini menggunakan fungsi `IO.puts/1` untuk memaparkan log di konsol kita.

Sebelum menjalankan semula fungsi dengan nombor bukan happy number, tambahkan baris ini di dalam badan fungsi `find/1`:

```elixir
IO.puts("The number is #{number}")
```
---

Kita akan mendapat paparan berikut apabila menjalankan semula 

```elixir
HappyNumberFinder.find(400)
```

```console
iex(2)> HappyNumberFinder.find(400)
The number is 400
The number is 16
The number is 37
The number is 58
The number is 89
The number is 145
The number is 42
The number is 20
The number is 4
The number is 16
The number is 37
The number is 58
The number is 89
The number is 145
The number is 42
The number is 20
The number is 4
```

Anda nampak corak di situ?  Pada satu peringkat, nombor-nombor mula berulang, dan corak sama akan berulang tanpa henti.  Dalam kata lain:*sebaik sahaja kita nampak satu cubaa bertindih, kita tahu nombor itu bukan happy number*, sebab kita tahu satu pertindihan bermaksud rekursi itu akan berterusan tanpa henti.

Ini hebat! Kita tahu bagaimana menyelesaikannya. Betul?

---

### Immutibility (Ketidakubahan)

Ini kali pertama ciri ketidakubahan data Elixir telah menyukarkan kita.  Bayangkan penyelesaian kita dalam bahasa seperti Javascript, yang memiliki state bolehubah.  Kita akan menyimpan satu array(kemungkinan pembolehubah global) cubaan-cubaan lepas dan menambahkan setiap cubaan baru kepada array ini.  Kemudian kita akan memeriksa jika nombor semasa ada dalam array tersebut dan hentikan rekursi jika dijumpai.

Tetapi kebolehubahan tidak ada dalam Elixir, jadi bagaimana kita selesaikan masalah ini?  Kita masih perlu hantarkan satu list cubaan-cubaan terdahulu kepada `find/1` supaya setiap kali kita jalankannya, fungsi itu akan memanggil satu salinanan cubaan-cubaan tersebut.

Ubahkan badan fungsi `find/1` supaya ia memanggil satu fungsi baru yang kita namakan `_find/2`, dan pendahkan semua kerja sedia-ada ke dalam fungsi tersebut.  Pertama kali kita memanggil `_find/2` dari dalam `find/1`, kita akan hantarkan satu list kosong kepadanya.

Ini membenarkan kita menyimpan nama `find/1` dan menyembunyikan perlaksanaan penghantaran senarai cubaan-cubaa.  Dan bagaimana kita hantarkan list baru tersebut?  Lihat baris 14 di bawah, di mana kita guna [sum | guesses] untuk mencipta satu list baru: kepala list itu ialah cubaan paling baru, dan ekornya ialah satu list cubaan-cubaan terdahulu.

Akhir sekali, kita boleh membuat `_find/2` sebagai fungsi peribadi dengan mentakrifnya dengan `defp`, dan bukannya `def`; ini memperkasakan tabiat peribadi kandungannya dan menguatkan penggunaan API awam kita hanya kepada fungsi `find/1`.

```elixir
defmodule HappyNumberFinder do

  # def get_sum(number)
 
  def find(number) do
    _find(number, [])
  end

  defp _find(number, guesses) do
    sum = get_sum(number)

    case sum do
      1 -> {:happy}
      _ -> _find(sum, [sum | guesses])
    end
  end
end
```
Sekarang kedua-dua fungsi `find/1` dan `_find/2` mempunyai akses kepada list cubaan-cubaan, langkah seterusnya ialah untuk memutuskan samada nombor semasa wujud di dalam list cubaan dan menghentikan rekursi.

---

### Meneroka Klausa Kawalan

Klausa Kawalan ialah satu ciri ELixir yang membenarkan kita mencipta pelbagai takrifan kepada fungsi-fungsi dan menghadkan perlaksanaan hanya di bawah keadaan-keadaan tertentu.

Dalam kes kita: jika nombor kita ialah sebahagian dari list cubaan, kita perlu hentikan rekursi.  Perhatikan klausa kawalan kita dalam baris 9 di bawah.

```elixir
defmodule HappyNumberFinder do
 
  # def get_sum(number)

  def find(number) do
    _find(number, [])
  end

  defp _find(number, guesses) where number in guesses do
    {:unhappy}
  end

  defp _find(number, guesses) do
    # …
  end
end
```
 
 Tambahkan takrifan fungsi ini kepada modul.  Sekarang terdapat dua takrifan untuk `_find/2`: satu untuk dilaksanakan jika nombor itu ada di dalam list cubaan, dan lagi satu untuk masa-masa lain.  Pandai, bukan?  Tapi apa akan berlaku bila kita cuba kompilkan kod ini?

```console
(ArgumentError) invalid args for operator “in”, it expects a compile time list or range on the right side when used in guard expressions, got: guesses
```

Untuk mengatasinya kita akan membuat refactor kepada fungsi `_find/2` untuk menggunakan satu case statement untuk memastikan jika nombor itu wujud di dalam list cubaan-cubaan lepas.  Saya juga mengambil peluang ini memindahkan beberapa logik ke dalam satu lagi fungsi sokongan, `compute_result/2`, supaya kita jelas tentang tujuan kita di dalam `_find/2`.

Berikut adalah kod mutakhir:

```elixir
defmodule HappyNumberFinder do

  def find(number) when number > 0 do
    _find(number, [])
  end

  defp get_sum(number) do
    number
    |> Integer.digits
    |> Enum.map(fn(x) -> x * x end)
    |> Enum.reduce(fn(x, total) -> x + total end)
  end

  defp _find(number, guesses) do
    case number in guesses do
      true -> {:unhappy}
      false -> compute_result(number, guesses)
    end
  end

  defp compute_result(number, guesses) do
    sum = get_sum(number)
    case sum do
      1    -> {:happy}
      _    -> _find(sum, [number | guesses])
    end
  end
end
```

Setiap kali fungsi `_find/2` dipanggil, kita periksa jika nombor semasa wujud di dalam list cubaan-cubaan.  Jika tidak, kita panggil `compute_result/2`, yang menjumlahkan semua gandaan nombor-nombor tersebut dan pastikan jika ianya happy number, atau jika perlu panggil `_find/2` lagi sekali.

Kita cuba dengan satu nombor happy number dan satu bukan nombor happy number dan lihat keputusannya.

```console
iex> HappyNumberFinder.find(49) 
{:happy}
iex> HappyNumberFinder.find(324) 
{:unhappy}

``` 

Siap! Terima kasih kerana membaca, dan saya berharap anda telah belajar sesuatu tentang Elixir.