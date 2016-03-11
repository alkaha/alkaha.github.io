---
layout: post
title: Bagaimana Saya Bermula Elixir - Jose Valim
---

- Artikel asal: [http://howistart.org/posts/elixir/1](http://howistart.org/posts/elixir/1)
- Penulis asal: [Jose Valim](https://twitter.com/josevalim)
- Ringkasan: Jose Valim adalah pencipta bahasa Elixir.  Di dalam artikel ini beliau membina sebuah aplikasi ringkas yang menunjukkan keupayaan Elixir.

---

## Portal

[Portal](http://en.wikipedia.org/wiki/Portal_(video_game)) ialah satu permainan yang mengandungi beberapa siri teka-teki yang perlu diselesaikan dengan memindahkan karakter pemain atau objek-objek ringkas dari satu lokasi ke lokasi lain.

Untuk 'teleport', pemain perlu menggunakan senapang Portal untuk menembak 'pintu' kepada satu dinding atau lantai.  Kita akan dipindahkan ke lokasi lain dengan memasuki 'pintu' tersebut.

![image1](http://alkaha.github.io/assets/portal-drop.jpeg)

Di dalam panduan ini kita akan menggunakan bahasa Elixir untuk membina 'portal' dengan menembak 'pintu-pintu' berbagai warna dan memindahka data melalui 'pintu-pintu' tersebut.  Kita juga akan belajar bagaimana untuk mengedarkan 'pintu-pintu' tersebut ke pelbagai mesin di dalam sistem rangkaian kita.

![image1](http://alkaha.github.io/assets/portal-list.jpeg)

Berikut ada apa yang akan kita pelajari:

- Sistem Kekerang Interaktif(interactive shell) Elixir
- Memulakan projek baru Elixir
- Pemadanan Corak(Pattern Matching)
- Menggunakan Agent untuk menguruskan keadaan semasa(state) aplikasi
- Menggunakan 'struct' untuk struktur data
- Penggunaan 'protocol' untuk membina fungsi tambahan kepada bahasa asal
- Penggunaan 'Supervision tree' dan 'Application'
- Mengagihkan 'node-node' Elixir

Mari kita mulakan!

## Pemasangan

Laman web Elixir menerangkan cara untuk memasang dan menyediakan penggunaan Elixir.  Ikuti arahan-arahan yang sediakan di dalam laman [Pemasangan Elixir](http://elixir-lang.org/install.html).

Pengguna Elixir banyak menggunakan terminal Operating System mereka; apabila pemasangan telah siap, beberapa executable baru akan siap untuk digunakan.  Salah satunya adalah `iex`.  Taipkan `iex` ke dalam terminal (atau `iex.bat` jika pengguna Windows) untuk bermula.

{% highlight bash %}
$ iex
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)
{% endhighlight %}

`iex` adalah untuk 'Interactive Elixir'. Di dalam `iex` kita boleh menaipkan apa-apa arahan dan akan mendapat paparan hasil arahan tersebut:
{% highlight bash %}
iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
iex> # gunakan simbol ini untuk membuat komen
nil
{% endhighlight %} 
Selain dari nombor dan rentetan('string') sebagaimana di atas, kita juga selalu menggunakan 'data type' berikut:
{% highlight bash %}
iex> :atom           # satu identifier (juga dikenali sebagai Symbol di dalam bahasa lain)
:atom
iex> [1, 2, "three"] # Lists (selalunya digunakan untuk memegang beberapa nilai dinamik)
[1, 2, "three"]
iex> {:ok, "value"}  # Tuples (selalunya digunakan untuk memegang beberapa nilai statik)
{:ok, "value"}
{% endhighlight %} 
Setelah aplikasi Portal kita siap, kita akan berupaya untuk menaip arahan-arahan berikut di dalam `iex`:
{% highlight bash %}
# Tembak dua pintu: satu orange, satu blue
iex(1)> Portal.shoot(:orange)
{:ok, #PID<0.72.0>}
iex(2)> Portal.shoot(:blue)
{:ok, #PID<0.74.0>}
# Mula memindah senarai [1, 2, 3, 4] dari pintu orange ke pintu blue
iex(3)> portal = Portal.transfer(:orange, :blue, [1, 2, 3, 4])
#Portal<
       :orange <=> :blue
  [1, 2, 3, 4] <=> []
>
# Sekarang apabila kita membuat arahan push_right, data akan pergi ke pintu blue
iex(4)> Portal.push_right(portal)
#Portal<
    :orange <=> :blue
  [1, 2, 3] <=> [4]
>
{% endhighlight %} 
Hebat, bukan?

## Projek pertama kita
Elixir didatangkan bersama satu tool bernama `Mix`. `Mix` adalah apa yang pengguna Elixir gunakan untuk membuat, mengkompil, dan menguji projek-projek.  Buat satu projek baru yang bernama `portal` menggunakan `mix`. Apabila membuat projek ini, kita akan menghantar arahan `--sup`, iaitu arahan untuk membina satu 'supervision tree'.  Kita akan meneroka lebih lanjut apa itu 'supervision tree' kemudian.  Untuk masa sekarang, cuma taipkan:
{% highlight bash %}
$ mix new portal --sup
{% endhighlight %} 
Arahan di atas akan membuat satu direktori baru bernama `portal` yang mengandungi beberapa fail.  Masuk ke dalam direktori `portal` dan taipkan `mix test` untuk membuat pengujian projek.
{% highlight bash %}
$ cd portal
$ mix test  
{% endhighlight %}
Bagus, sekarang kita sudah mempunyai satu direktori dan 'test suite' siap untuk digunakan.

Sekarang kita akan lihat projek yang kita janakan tadi melalui aplikasi penyunting teks('text editor').  Saya tidak begitu memberi perhatian kepada aplikasi penyunting teks, saya selalunya menggunakan [Sublime Text 3](http://www.sublimetext.com/3) tetapi anda boleh mendapat [sokongan Elixir untuk beberapa aplikasi penyunting teks di laman ini](http://elixir-lang.org) di bawah 'Code Editor Support'.

Apabila aplikasi penyunting teks telah dibuka, lihat apa yang ada di dalam direktori-direktori berikut:

- _build - tempat Mix menyimpan artifak kompilasi
- config - tempat kita membuat konfigurasi kepada projek dan komponen sokongan('dependencies')
- lib - di mana kita menyimpan kod yang akan kita tulis
- mix.exs - di mana kita meletakkan nama projek, versi dan 'dependencies'
- test - di mana kita meletakkan fail-fail untuk tujuan pengujian('test')

Sekarang kita boleh menjalankan sesi `iex` di dalam projek kita, dengan menaip:
{% highlight bash %}
$ iex -S mix
{% endhighlight %}

## Pemadanan Corak(Pattern Matching)

Sebelum kita membina aplikasi, kita perlu belajar mengenai pemadanan corak('pattern matching').  Penggunaan `=` di dalam Elixir agak berbeza dengan apa yang kita lihat di dalam bahasa lain:
{% highlight bash %}
iex> x = 1
1
iex> x
1
{% endhighlight %}
Nampak macam biasa, tapi apa akan jadi jika kita terbalikkan operasi itu:
{% highlight bash %}
iex> 1 = x
1
{% endhighlight %}
Berjaya!  Ini kerana Elixir cuba untuk menyesuaikan arahan sebelah kanan dengan arahan di sebelah kiri.  Oleh sebab keduanya di-set kepada `1`, arahan tersebut berjaya dijalankan.  Kita cuba yang lain pula:
{% highlight bash %}
iex> 2 = x
** (MatchError) no match of right hand side value: 1
{% endhighlight %}
Sekarang kenyataan ini tidak dapat disesuaikan, jadi kita akan mendapat ralat.  Di dalam Elixir, kita juga gunakan kaedah pemadanan corak('pattern matching') untuk penyesuaian struktur data.  Sebagai contoh, kita boleh gunakan `[head|tail]` untuk mengekstrak 'head'(elemen pertama) dan 'tail'(elemen selebihnya) di dalam satu List:
{% highlight bash %}
iex> [head|tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
{% endhighlight %}
Menyesuaikan list kosong kepada [head\|tail] menyebabkan ralat:
{% highlight bash %}
iex> [head|tail] = []
** (MatchError) no match of right hand side value: []
{% endhighlight %}
Akhir sekali kita juga boleh gunakan kenyataan [head\|tail] untuk menambah satu elemen kepada kepala satu list:
{% highlight bash %}
iex> list = [1,2,3]
[1,2,3]
iex> [0|list]
[0,1,2,3] 
{% endhighlight %}

## Membuat model pintu portal menggunakan Agent

Data struktur Elixir adalah 'immutable'(tidak membenarkan perubahan pada data setelah diset).  Di dalam contoh di atas, kita tidak membuat perubahan kepada list tersebut.  Kita boleh ceraikan satu list, atau menambah elemen baru, tetapi list asal tidak akan berubah.

Walaupun begitu, kita perlu menyimpan bentuk keadaan data(state), contohnya kedudukan data semasa pindahan melalui portal, kita perlukan kaedah untuk menyimpan keadaan semasa data semasa proses tersebut.  Di dalam Elixir, salah satu kaedah tersebut dipanggil sebagai 'agent'.  Sebelum meggunakan 'agent' kita perlu melihat secara ringkas akan fungsi tanpa nama('anonymous function'):
{% highlight bash %}
iex> adder = fn a,b -> a + b end
#fungsi<12.90072148/2 in :er_eval:expr/5>
iex> adder.(1,2)
3
{% endhighlight %}
Satu fungsi tanpa nama('anonymous function') dimulai dengan `fn` dan diakhiri dengan `end`, dan simbol `->` digunakan utuk mengasingkan argumen dari badan fungsi tersebut.  Kita gunakan fungsi tanpa nama('anonymous function') untuk mengasakan('initialize'), mencapai dan mengemaskini sesatu 'agent'. 
{% highlight bash %}
iex> {:ok, agent} = Agent.start_link(fn -> [] end)
{:ok, #PID<0.61.0>}
iex> Agent.get(agent, fn list -> list end)
[]
iex> Agent.update(agent, fn list -> [0|list] end)
:ok
iex> Agent.get(agent, fn list -> list end)
[0]
{% endhighlight %}
> Nota: anda mungkin akan mendapat nilai #PID<..> berlainan dari contoh di atas, ianya sesuatu yang memang dijangkakan.

Di dalam contoh di atas, kita membuat satu 'agent' baru, menghantar satu fungsi yang menghasilkan keadaan awal, iaitu satu list kosong([]).  'Agent' tersebut memulangkan {:ok, #PID<0.61.0>}

Simbol `{}` di dalam Elixir menandakan sebagai `tuple`, yang di dalam contoh di atas mengandungi `atom` `:ok` dan 'process identfier(PID)'.  Kita gunakan atom di dalam Elixir sebagai tag(penanda).  Di dalam contoh di atas, kita menandakan 'agent' tersebut berjaya dijalankan.

`#PID<..>` pula adalah pengenal pasti proses('process identifier') kepada 'agent' tersebut. Apabila kita menyebut proses di dalam Elixir, kita tidak maksudkan proses-proses sistem pengendalian('operating system'), tetapi proses-proses Elixir yang ringan dan wujud secara berasingan, memberikan peluang untuk menjalankan beratus ribu proses-proses tersebut di dalam mesin yang sama.

Kita simpankan PID 'agent' di dalam pembolehubah `agent`, yang membenarkan kita menghantar mesej-mesej untuk mendapatkan dan mengemaskini keadaan semasa 'agent' tersebut.

Kita akan gunakan 'agent' untuk membina pintu-pintu portal kita.  Buat satu fail bernama `lib/portal/door.ex` yang mempunyai kandungan berikut:
{% highlight ruby %}
defmodule Portal.Door do
  @doc """
  Starts a door with the given `color`.
  The color is given as a name so we can identify
  the door by color name instead of using a PID.
  """
  def start_link(color) do
    Agent.start_link(fn -> [] end, name: color)
  end
  @doc """
  Get the data currently in the `door`.
  """
  def get(door) do
    Agent.get(door, fn list -> list end)
  end
  @doc """
  Pushes `value` into the door.
  """
  def push(door, value) do
    Agent.update(door, fn list -> [value|list] end)
  end
  @doc """
  Pops a value from the `door`.
  Returns `{:ok, value}` if there is a value
  or `:error` if the hole is currently empty.
  """
  def pop(door) do
    Agent.get_and_update(door, fn
      []    -> { :error, [] }
      [h|t] -> { { :ok, h }, t }
    end)
  end
end
{% endhighlight %}
Di dalam Elixir, kita menulis kod di dalam modul-modul, yang pada asasnya adalah fail-fail yang mengandungi beberapa kumpulan fungsi.  Di atas, kita telah menulis empat fungsi, semuanya siap didokumentasikan.

Sekarang kita akan menguji kod yang kita tulis.  Buka satu shell baru dengan menaip `iex -S mix`.  Apabila memulakan satu sesi shell yang baru, Elixir akan mengkompilkan fail kita, jadi kita boleh terus menggunakannya:
{% highlight bash %}
iex> Portal.Door.start_link(:pink)
{:ok, #PID<0.68.0>}
iex> Portal.Door.get(:pink)
[]
iex> Portal.Door.push(:pink, 1)
:ok
iex> Portal.Door.get(:pink)
[1]
iex> Portal.Door.pop(:pink)
{:ok, 1}
iex> Portal.Door.get(:pink)
[]
iex> Portal.Door.pop(:pink)
:error
{% endhighlight %}
Excellent!

Satu aspek yang menarik mengenai Elixir ialah di mana dokumentasi diberikan layanan utama.  Oleh sebab kita telah siap menyediakan dokumetasi untuk kod Portal.Door, kita boleh mencapai dokumentasi tersebut melalui terminal.  Cuba:
{% highlight bash %}
iex> h Portal.Door.start_link
{% endhighlight %}
## pindahan Melalui Portal

Pintu portal kita kini telah tersedia dan telah sampai masanya untuk kita mula membina fungsi pindahan melalui portal!  Untuk menyimpan data semasa proses ini, kita akan membuat satu struktur data 'struct' dipanggil `Portal`.  Kita cuba dahulu apa itu 'struct' di dalam `iex` sebelum meneruskan:
{% highlight ruby %}
iex> defmodule User do
...>   defstruct [:name, :age]
...> end
iex> user = %User{name: "john doe", age: 27}
%User{name: "john doe", age: 27}
iex> user.name
"john doe"
iex> %User{age: age} = user
%User{name: "john doe", age: 27}
iex> age
27
{% endhighlight %}
'Struct' diisytiharkan di dalam modul dan menggunakan nama modul tersebut.  Selepas 'struct' diisytiharkan, kita boleh menggunakan sintaks `%User{...}` untuk membina 'struct' lain atau menyesuaikan antara satu sama lain.

Bukakan fail `lib/portal.ex` dan tambah beberapa kod ke dalam modul `Portal`.  Kita akan dapati modul ini telah diisi dengan satu fungsi yang bernama `start/2`.  Jangan buang fungsi ini, kita akan bincangkan ia di bahagian seterusnya, buat masa ini kita cuma perlu menambah kod-kod baru di dalam modul `Portal` tersebut:
{% highlight ruby %}
# isytiharkan 'struct'. 'Struct' ini akan diakses dengan sintaks %Portal{:left, :right}
defstruct [:left, :right]

@doc """
Starts transfering `data` from `left` to `right`.
"""
def transfer(left, right, data) do
  # First add all data to the portal on the left
  for item <- data do
    Portal.Door.push(left, item)
  end

  # Returns a portal struct we will use next
  %Portal{left: left, right: right}
end

@doc """
Pushes data to the right in the given `portal`.
"""
def push_right(portal) do
  # See if we can pop data from left. If so, push the
  # popped data to the right. Otherwise, do nothing.
  case Portal.Door.pop(portal.left) do
    :error   -> :ok
    {:ok, h} -> Portal.Door.push(portal.right, h)
  end

  # Let's return the portal itself
  portal
end

We have defined our Portal struct and a Portal.transfer/3 fungsi (the /3 indicates the fungsi expects three arguments). Let's give this transfer a try. Start another shell with iex -S mix so our changes are compiled and type:

# Start doors
iex> Portal.Door.start_link(:orange)
{:ok, #PID<0.59.0>}
iex> Portal.Door.start_link(:blue)
{:ok, #PID<0.61.0>}

# Start transfer
iex> portal = Portal.transfer(:orange, :blue, [1, 2, 3])
%Portal{left: :orange, right: :blue}

# Check there is data on the orange/left door
iex> Portal.Door.get(:orange)
[3, 2, 1]

# Push right once
iex> Portal.push_right(portal)
%Portal{left: :orange, right: :blue}

# See changes
iex> Portal.Door.get(:orange)
[2, 1]
iex> Portal.Door.get(:blue)
[3]
{% endhighlight %}
Nampaknya pindahan data melalui portal berjalan sebagai yang dijangka. Perhatikan bahawa data adalah di dalam susunan songsang dari susunan data yang dipindahkan.  Ini dijangkakan kerana kita mahukan elemen terakhir dari list(dalam kes ini nombor 3) mejadi elemen pertama yang dihantar melalui pintu di sebelah kanan(blue).

Buat masa ini, data yang dipaparkan oleh `iex` adalah dalam bentuk 'struct', iaitu `%Portal{left: :orange, right: :blue}`. Kita mahukan paparan yang menunjukkan 'perjalanan' data semasa proses pindahan.

Kita akan bina fungsi ini seterusnya.

## Penggunaan Protocols

Kita telah tahu yang kita boleh memaparkan data di dalam `iex`.  Yakni, jika kita menaip `1 + 2`, `iexe` akan memaparkan `3`.  Adakah kita dibenarkan untuk membuat perubahan kepada bentuk data yang dipaparkan?

Ya, boleh!  Elixir menyediakan 'protocol', yang membenarkan kita untuk membuat perubahan kepada kelakuan asal dan boleh digunakan ke atas semua jenis data.

Sebagai contoh, setiap kali sesuatu dipaparkan di dalam terminal `iex`, Elixir menggunakan 'protocol' `Inspect`.  Oleh sebab 'protocol' boleh diubahsuai untuk semua jenis data pada setiap masa, ia juga boleh digunapakai untuk `Portal`.  Buka fail `lib/portal.ex` dan pada penghujung fail, di luar modul `Portal`, tambahkan kod berikut:
{% highlight ruby %}
defimpl Inspect, for: Portal do
  def inspect(%Portal{left: left, right: right}, _) do
    left_door  = inspect(left)
    right_door = inspect(right)

    left_data  = inspect(Enum.reverse(Portal.Door.get(left)))
    right_data = inspect(Portal.Door.get(right))

    max = max(String.length(left_door), String.length(left_data))

    """
    #Portal<
      #{String.rjust(left_door, max)} <=> #{right_door}
      #{String.rjust(left_data, max)} <=> #{right_data}
    >
    """
  end
end
{% endhighlight %}
Di dalam 'snippet' di atas kita telah membina 'protocol' `Inspect` untuk 'struct' `Portal`.  'Protocol' ini cuma memerlukan satu fungsi yang dinamakan `inspect` dan memerlukan dua argumen, iaitu 'struct' `Portal` itu sendiri dan satu argumen lain yang kita akan biarkan dahulu buat masa ini.

Kemudian kita akan memanggil `inspect` beberapa kali, untuk mendapatkan struktur dan data untuk `left` dan `right`.  Akhir sekali kita akan memulangkan satu rentetan('string') yang menggambarkan kedudukan portal tersebut.

Mulakan satu sesi baru `iex` dengan menaip `iex -S mix` untuk melihat hasilnya:
{% highlight bash %}
iex> Portal.Door.start_link(:orange)
{:ok, #PID<0.59.0>}
iex> Portal.Door.start_link(:blue)
{:ok, #PID<0.61.0>}
iex> portal = Portal.transfer(:orange, :blue, [1, 2, 3])
#Portal<
    :orange <=> :blue
  [1, 2, 3] <=> []
>
{% endhighlight %}

## Proses Menembak Pintu Portal

Kita selalu dengar bahawa Erlang VM, iaitu 'virtual machine' yang digunakan oleh Elixir, adalah amat bagus untuk membina aplikasi yang bersifat tahan rosak('fault tolerant').  Salah satu faktor yang membenarkan keadaan ini adalah sesuatu yang dipanggil sebagai 'supervision trees'.

Kod kita setakat ini adalah tidak di'urus'kan('supervised').  Cuba kita lihat apa yang berlaku apabila kita mematikan salah satu 'agent' pintu portal tersebut:
{% highlight ruby %}
# Start doors and transfer
iex> Portal.Door.start_link(:orange)
{:ok, #PID<0.59.0>}
iex> Portal.Door.start_link(:blue)
{:ok, #PID<0.61.0>}
iex> portal = Portal.transfer(:orange, :blue, [1, 2, 3])

# First unlink the door from the shell to avoid the shell from crashing
iex> Process.unlink(Process.whereis(:blue))
true
# Send a shutdown exit signal to the blue agent
iex> Process.exit(Process.whereis(:blue), :shutdown)
true

# Try to move data
iex> Portal.push_right(portal)
** (exit) exited in: :gen_server.call(:blue, ..., 5000)
    ** (EXIT) no process
    (stdlib) gen_server.erl:190: :gen_server.call/3
    (portal) lib/portal.ex:25: Portal.push_right/1
{% endhighlight %}
Kita mendapat 'exit error' kerana pintu 'blue' tidak lagi wujud.  Untuk mengatasi ralat ini,  kita akan membina satu 'supervisor' yang akan bertanggugjawab untuk menghidupkan semula pintu portal yang dimatikan secara mengejut.

Jika kita ingat, kita telah menggunakan `--sup` ketika mula membuat projek ini.  Kita gunakan 'flag' tersebut sebab selalunya 'supervisor' berfungsi di dalam satu 'supervision tree' dan 'supervision tree' akan dijalankan sebagai sebahagian dari aplikasi.  Apa yang `--sup` lakukan adalah membina struktur 'supervision' secara lalai('default'), dan dapat kita lihat di dalam modul `Portal` kita:
{% highlight ruby %}
defmodule Portal do
  use Application

  # See http://elixir-lang.org/docs/stable/elixir/Application.html
  # for more information on OTP Applications
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # Define workers and child supervisors to be supervised
      # worker(Portal.Worker, [arg1, arg2, arg3])
    ]

    # See http://elixir-lang.org/docs/stable/elixir/Supervisor.html
    # for other strategies and supported options
    opts = [strategy: :one_for_one, name: Portal.Supervisor]
    Supervisor.start_link(children, opts)
  end

  # ... fungsis we have added ...
end
{% endhighlight %}
Kod di atas menjadikan modul `Portal` sebagai 'application callback'.  'Application callback' mesti menyediakan satu fungsi bernama `start/2`, seperti yang kita lihat di atas, dan fungsi ini mesti menjalankan satu 'supervisor' sebagai 'root' kepada 'supervision tree' kita.  Sekarang ini 'supervision tree' kita tidak mempunyai 'children' dan ini yang akan kita ubah seterusnya.

Ubah fungsi `start/2` di atas kepada berikut:
{% highlight ruby %}
def start(_type, _args) do
  import Supervisor.Spec, warn: false

  children = [
    worker(Portal.Door, [])
  ]

  opts = [strategy: :simple_one_for_one, name: Portal.Supervisor]
  Supervisor.start_link(children, opts)
end
{% endhighlight %} 
Kita telah membuat dua perubahan:

- Kita telah menambah satu 'child' kepada 'supervisor' kita, iaitu dari jenis `worker`, dan 'child' tersebut adalah dari modul `Portal.Door`.  Kita tidak meghantar apa-apa argumen kepada 'worker' tersebut, cuma satu list kosong(`[]`).
- Kita juga mengubah startegi daripada `:one_for_one` kepada `:simple_one_for_one`.  'Supervisor' menyediakan pelbagai strategi dan `:simple_one_for_one` adalah berguna apabila kita mahu membuat 'children' secara dinamik, selalunya dengan argumen yang berbeza.  Ini adalah apa yang kita perlukan untuk pintu-pintu portal kita, di mana kita mahu mewujudkan pelbagai pintu dengan pelbagai warna.

Langkah terakhir ialah untuk menambah satu fungsi bernama `shoot/1` ke dalam modul `Portal` yang akan menerima argumen 'color' dan akan mewujudkan satu pintu portal baru sebagai sebahagian dari 'supervision tree':
{% highlight ruby %}
@doc """
Shoots a new door with the given `color`.
"""
def shoot(color) do
  Supervisor.start_child(Portal.Supervisor, [color])
end
{% endhighlight %}
fungsi di atas menjadi sebahagian dari 'supervisor' bernama `Portal.Supervisor` dan meminta supaya satu proses 'child' dimulakan.  `Portal.Supervisor` adalah nama kepada 'supervisor' yang diisytiharkan di dalam fungsi `start/2` dan proses 'child' adalah `Portal.Door` yang diklasifikasikan sebagai 'worker' kepada 'supervisor' tersebut.

Untuk memulakan proses 'child' tersebut, 'supervisor' perlu memulakan fungsi `Portal.Door.start_link(color)`, di mana 'color' adalah nilai yang dihantar kepada fungsi `start/2` di atas.  Jika kita memulakan fungsi `Supervisor.start_child(Portal.Supervisor, [foo, bar, baz])`, 'supervisor' itu akan cuba untuk menjalankan proses 'child' melalui `Portal.Door.start_link(foo, bar, baz)`.

Masa untuk menguji fungsi 'menembak' pintu portal kita.  Mulakan sesi `iex -S mix` dan:
{% highlight bash %}
iex> Portal.shoot(:orange)
{:ok, #PID<0.72.0>}
iex> Portal.shoot(:blue)
{:ok, #PID<0.74.0>}
iex> portal = Portal.transfer(:orange, :blue, [1, 2, 3, 4])
#Portal<
       :orange <=> :blue
  [1, 2, 3, 4] <=> []
>

iex> Portal.push_right(portal)
#Portal<
    :orange <=> :blue
  [1, 2, 3] <=> [4]
>
{% endhighlight %}

Apa akan jadi jika kita matikan proses untuk `:blue` sekarang?
{% highlight bash %}
iex> Process.unlink(Process.whereis(:blue))
true
iex> Process.exit(Process.whereis(:blue), :shutdown)
true
iex> Portal.push_right(portal)
#Portal<
  :orange <=> :blue
   [1, 2] <=> [3]
>
{% endhighlight %}
Perhatikan pada masa ini operasi `push_right/1` dapat dijalankan kerana 'supervisor' akan menghidupkan secara otomatik satu lagi proses untuk `:blue`, setelah kita mematikan proses yang asal.  Malangnya data yang berada di dalam proses `:blue` yang asal hilang semasa proses itu dimatikan, tetapi sekurang-kurangnya sistem kita berjaya untuk pulih dari 'crash' tersebut.

Terdapat pelbagai strategi untuk 'supervision', dan juga mekanisma untuk menyimpan data walaupun jika sesuatu yang tidak diduga berlaku, memberikan anda pilihan untuk memilih strategi terbaik untuk aplikasi anda.

Outstanding!

## Pindahan Teragih(Distributed transfers)

Aplikasi portal kita telah berjaya dijalankan, jadi sekarang kita akan mencuba proses pindahan teragih('distributed transfer').  Ianya amat hebat jika anda menjalankan kod kita di atas dua mesin berlainan di dalam rangkaian anda.  Walaubagaimanapun, jika tidak mempunya mesin lain, kita masih boleh membuat pengujian ini.

Kita boleh mulakan satu sesi `iex` sebagai satu nod di dalam satu 'network' dengan menggunakan pilihan `--sname`.  Kita boleh mula mencuba:
{% highlight bash %}
$ iex --sname room1 --cookie secret -S mix
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)
iex(room1@jv)1>
{% endhighlight %}
Kita boleh lihat di dalam terminal `iex` ini agak berlainan dengan yang sebelum ini.  Sekarang kita nampak nama `room1@jv` dipaparkan.  `room1` adalah nama yang kita berikan untuk nod dan `jv` adalah 'network name' kepada komputer yang menjalankan nod tersebut.  Mesin saya menunjukkan `jv`, mesin anda mungkin menggunakan nama lain.  Kod di bawah akan menggunakan `room1@COMPUTER-NAME` dan `room2@COMPUTER-NAME` dan anda perlu menukar `COMPUTER-NAME` kepada 'network name' komputer anda.

Di dalam sesi `iex` bernama `room1`, cuba 'tembak' satu pintu `:blue`:
{% highlight bash %}
iex(room1@COMPUTER-NAME)> Portal.shoot(:blue)
{:ok, #PID<0.65.0>}
{% endhighlight %}  
Sekarang mulakan satu sesi `iex` bernama `room2`:
{% highlight bash %}
$ iex --sname room2 --cookie secret -S mix
{% endhighlight %}
> Nota: 'cookie' yang sama hendaklah digunakan di kedua-dua komputer untuk membenarkan kedua-dua nod Elixir tersebut berkomuikasi antara satu sama lain.

API 'Agent' membenarkan kita untuk membuat permohonan silang nod('cross-node requests').  Apa yang perlu dilakukan adalah menghantar nama nod di mana 'agent' yang cuba dicapai sedang berjalan apabila menjalankan fungsi-fungsi `Portal.Door`.  Sebagai contoh cuba capai pintu 'blue' dari `room2`:
{% highlight bash %}
iex(room2@COMPUTER-NAME)> Portal.Door.get({:blue, :"room1@COMPUTER-NAME"})
[]
{% endhighlight %}
Ini bermaksud kita boleh dapat melaksanakan pindahan teragih('distributed transfer') hanya dengan menggunakan nama nod.  Ketika masih di dalam `room2`, cuba:
{% highlight bash %}
iex(room2@COMPUTER-NAME)> Portal.shoot(:orange)
{:ok, #PID<0.71.0>}
iex(room2@COMPUTER-NAME)> orange = {:orange, :"room2@COMPUTER-NAME"}
{:orange, :"room2@COMPUTER-NAME"}
iex(room2@COMPUTER-NAME)> blue = {:blue, :"room1@COMPUTER-NAME"}
{:blue, :"room1@COMPUTER-NAME"}
iex(room2@COMPUTER-NAME)> portal = Portal.transfer(orange, blue, [1, 2, 3, 4])
#Portal<
  {:orange, :room2@COMPUTER-NAME} <=> {:blue, :room1@COMPUTER-NAME}
          [1, 2, 3, 4] <=> []
>
iex(room2@COMPUTER-NAME)> Portal.push_right(portal)
#Portal<
  {:orange, :room2@COMPUTER-NAME} <=> {:blue, :room1@COMPUTER-NAME}
             [1, 2, 3] <=> [4]
>
{% endhighlight %}
Awesome!  Kita dapat melaksanakan pindahan teragih('distributed transfer') tanpa mengubah walau satu baris dari kod kita.

Walaupun masa ini `room2` sedang membuat koordinasi proses pindahan, kita masih boleh melihat proses pindahan tersebut dari `room1`.
{% highlight bash %}
iex(room1@COMPUTER-NAME)> orange = {:orange, :"room2@COMPUTER-NAME"}
{:orange, :"room2@COMPUTER-NAME"}
iex(room1@COMPUTER-NAME)> blue = {:blue, :"room1@COMPUTER-NAME"}
{:blue, :"room1@COMPUTER-NAME"}
iex(room1@COMPUTER-NAME)> Portal.Door.get(orange)
[3, 2, 1]
iex(room1@COMPUTER-NAME)> Portal.Door.get(blue)
[4]
{% endhighlight %}
Perlaksanaan pindahan teragih('distributed transfer') untuk portal kita berjaya dijalankan sebab pintu-pintu portal hanyalah proses-proses dan mencapai/memindah data melalui pintu-pintu tersebut dilakukan dengan memindah mesej-mesej kepada proses-proses tersebut melalui 'Agent API'.  Kita sebut pindahan mesej di dalam Elixir sebagai 'location transparent': kita boeh memindah mesej kepada mana-mana PID tidak kira jika proses-proses tersebut sedang berjalan di dalam nod komputer yang sama atau nod yang berada dalam komputer lain di dalam satu rangkaian yang sama.

## Penutup

Kita telah sampai ke penghujung panduan bagaimana untuk bermula dengan Elixir!  Dengan pantasnya kita telah bermula dengan memulakan proses membuat pintu portal secara manual kepada membuatnya sebagai tahan rosak('fault tolerant') untuk kegunaan dalam melaksanakan pindahan teragih('distributed transfer')!

Kami mencabar anda untuk meneruskan mempelajari dan meneroka Elixir dengan menambahbaik aplikasi portal ini:

- Menambah fungsi `Portal.push_left/1` yang membuat pindahan data dari pintu kanan ke pintu kiri.  Bagaimana untuk megelak dari menulis kod berulang di dalam kedua-dua fungsi `push_left/1` dan `push_right/1` tersebut?
- Belajar mengenai [ExUnit](http://elixir-lang.org/docs/stable/ex_unit/ExUnit.html), 'testing framework' untuk Elixir, dan menulis 'test' kepada implementasi kita setakat ini.  Kita sudahpun mempunyai struktur asas untuk ini di dalam direktori `test`.
- Menjana dokumentasi dalam bentuk HTML menggunakan [ExDoc](http://github.com/elixir-lang/ex_doc).
- Menyiarkan projek anda ke sumber luaran, seperti [Github](https://github.com/), dan menerbitkan satu pakej menggunakan [Hex Package Manager](https://hex.pm/).

Kami mengalu-alukan anda untuk meneroka [laman web kami](http://elixir-lang.org/) dan baca panduan Getting Started atau lain-lain sumber yang telah kami bekalkan untuk anda mempelajari lebih lanjut mengenai Elixir dan komuniti kami.

Akhir sekali, berbanyak terima kasih kepada [Augie De Blieck Jr.](http://twitter.com/augiedb) yag menyediakan lukisan di dalam tutorial ini.

See you around! 














