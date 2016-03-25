---
layout: post
title: Aturcara Kefungsian Bukan Pelik - anda cuma perlukan beberapa corak baru
---

- Dokumen Asal: [Functional Programming is not weird: you just need some new patterns](https://medium.com/@cameronp/functional-programming-is-not-weird-you-just-need-some-new-patterns-7a9bf9dc2f77#.57grdaho2)
- Penulis Asal: [Cameron Price](https://medium.com/@cameronp)

---

Saya nampak sesuatu di Hacker News hujung minggu ini yang setahun lalu akan saya persetujui, tetapi tidak lagi: [Functional Programming Is Not Popular Because It Is Weird](http://probablydance.com/2016/02/27/functional-programming-is-not-popular-because-it-is-weird/).

> Menulis kod kefungsian selalunya kolot dan dirasai lebih sebagai menyelesaikan teka-teki daripada cuba menjelaskan satu proses kepada komputer.  Dalam bahasa-bahasa kefungsian saya selalunya tahu apa yang mahu saya kata, tetapi selalunya rasa seperti perlu menyelesai teka-teki untuk menjelaskannya kepada bahasa tersebut.

Ungkapan ini betul-betul menarik perhatian saya.  Semasa mempelajari Elixir, dan sebelumnya Scala, saya selalu rasa saya menghabiskan satu jam mengkaji satu masalah, otak berpusing-pusing untuk mencuba dan membayangkan bagaimana untuk memodelkannya tanpa memanipulasi state.  Kemudian saya akan akhirnya berjaya mencapai penyelesaian, yang rupa-rupanya cuma tiga baris kod yang cantik, elegan dan berfungsi dalam percubaan pertama.  Ia dirasakan lebih seperti bermain dari apa-apa yang saya bayangkan sebagai kerja serius(Serious Work).

![xkcd-1270](/assets/xkcd-1270.png)[Ini saya setahun lalu. Gambar oleh: xkcd.com](https://xkcd.com/1270/)

Tapi saya seronok dengan permainan itu, dan apabila saya mula memahami faedah-faedah 'backend' Erlang, dalam bentuk konkurensi, saya mengambil keputusan untuk teruskan dengannya.  Dan sesuatu yang tidak dijangka berlaku.  Saya telah menyelesaikan beberapa masalah kecil untuk beberapa ketika.  [Projek Euler](https://projecteuler.net/), masalah-masalah di dalam [Programming Elixir oleh Dave Thomas](https://pragprog.com/book/elixir12/programming-elixir-1-2), dan lain-lain.  Sampai satu ketika, saya perasan bahawa saya tidak lagi "menyelesaikan teka-teki" untuk mengeluarkan hasil-hasil mudah, saya sebenarnya menulis kod untuk menyelesaikan masalah dalam kadar lebih cepat, dan dengan kepersisan lebih tinggi dari sebelum ini.  Algoritma-algoritma rumit dirasakan seperti mengalir keluar dari saya, dan mereka mengujakan saya dengan berfungsi betul sedari mula.  Ianya seperti segala janji-janji oleh penyokong-penyokong aturcara kefungsian dipenuhi sekaligus.  Pelik.


Saya merasa pelik kenapa ianya terjadi.  Adakah tiba-tiba saya mendapat kebolehan untuk berfikir di dalam cara tunggang terbalik sebagaimana yang diminta oleh aturcara kefungsian?  Memang benar saya lebih pantas, dan mungkin berbunyi seperti seorang fanatik, perasaannya memang mengujakan.  Jadi saya berfikir apa sebenarnya yang berlaku, dan bagaimana saya boleh meneruskannya, dan menolong orang lain pada masa yang sama.


Ianya mengenai corak.  Bukannya corak seni bina besar gaya-GOF yang terlalu banyak dibincangkan, tetapi corak-corak mudah yang digunakan sehari-hari apabila menulis kod.  Mereka terlalu kecil sehiggakan selalunya tidak dinamakan, tetapi tidak kira jika anda pengaturcara kefungsian atau pengaturcara imperatif, anda selalu menggunakannya.  Sesetengahnya spesifik untuk bahasa-bahasa tertentu, sesetengah lagi corak-corak lazim dalam satu keluarga bahasa.  Jika anda telah menulis kod imperatif selama ini, anda memiliki satu peti alatan yang besar yang boleh dicapai bila-bila masa, dan tanpa perlu berfikir tentang mereka.  Contohnya:

```c
void print_arr(int *list, int count) {
  for(int i = 0; i <= count; i++) {
    printf(“%d “, list[i] );
  }
  printf(“\n”);
}
```

Anda nampak pepijat di dalam kod di atas?  Jika anda pernah menggunakan C, atau Java, atau bahasa-bahasa lain sepertinya, anda terus tahu apa yang sepatutnya dilakukan kod ini, dan, anda terus tahu bahawa ianya sepatutnya `i < count`, bukan `i <= count`.  Anda keluarkan gelungan seperti ini tanpa perlu berfikir, tapi saya berani bertaruh bahawa beberapa tahun lalu, semasa anda baru belajar, anda perlu berfikir tentang gelungan seperti ini.  Tetapi sekarang corak-corak ini boleh anda keluarkan bila-bila masa tanpa perlu 'menjalankan array', atau 'menjalankan string', dan anda mengenalinya apabila dilihat tanpa perlu berfikir dalam mengenai bagaimana ia berfungsi.


Dalam era 90-an, semasa saya menghabiskan banyak masa menemu duga pengaturcara C, satu soalan temuduga lazim ialah "tuliskan satu fungsi untuk menyingkirkan pertindihan daripada satu senarai tersusun integer".  Saya amat menyukai soalan ini, dan saya menyoalnya berulang-kali.  Saya mungkin telah menyoal 200 orang untuk menyelesaikan masalah ini, sebab pada masa itu, ianya cara hebat untuk melihat keselesaan calon-calon dengan C.(Saya tahu, saya tahu, temuduga papan putih bukan ujian yang baik, kita tidak menulis kod atas papan putih, dan lain-lain.  Berilah saya peluang, ianya 90-an.  Kami lakukan banyak perkara yang mengerikan anda hari ini.)


Apa yang saya sukakan tentang apa yang kami panggil sebagai "sorted-array-with-dupers", adalah 99% calon boleh menjelaskan strategi asas bagaimana mereka akan menyelesaikannya:

1.  Jalankan array..
2.  awasi nilai paling semasa yang telah anda ambil..
3.  dan jika ia berubah, ambil nilai baru, dan tetapkan nilai itu sebagai nilai semasa kita
4.  sehingga sampai ke hujung, dan kemudian pulangkan array baru itu.


Semua orang sampai ke tahap ini, tetapi cuma 10-15% calon-calon yang berjaya menterjemahkannya ke dalam bentuk kod.  Ini kerana ia melibatkan dua iterator, dan mengawal kedua-dua iterator itu adalah corak yang memerlukan pengalaman.  Ia juga jenis corak yang akan dijumpai berulang-kali apabila mengaturcara C.


Ini kod yang mungkin nampak dalam C:

```c
int rem_dupes(int *array, int count) {
 if (count == 0)
   return 0;
 int current_val = array[0];
 int current_output_position = 1;
 for (int i = 1; i < count; i++) {
   if (array[i] != current_val) {
     array[current_output_position++] = array[i];
     current_val = array[i];
   }
  }
  return(current_output_position);
}
```    


Selepas menguruskan kes sisi `count == 0`, ia menjalankan array dengan gelungan utama `for`, dan  mengemaskini nilai semasa iterator kedua (current_output_position) secara manual.  Kemudian ia memulangkan nilai hitung yang baru.  Jika anda biasa menulis C, atau Java, anda mungkin dapat grok kod di atas tanpa masalah.


Tapi saya rasa anda akan bersetuju dengan saya bahawa itu bukanlah perkara paling mudah dalam dunia, dan satu masa dahulu anda mungkin telah bersusah-payah untuk menulis kod tersebut tanpa sebarang pepijat.  Sekarang pun, anda yakinkah tidak ada sebarang pepijat di dalam kod tersebut?  Saya tidak 100% yakin, dan saya telah kompilkan dan jalankannya pagi tadi.


Berikut adalah kod yang sama, ditulis dalam bentuk rekursif, dalam Elixir menggunakan fungsi pelbagai kepala dan pemadanan corak:

```elixir
def rem_dupes([]), do: []
def rem_dupes([first | t]), do: [first | rem_dupes(t, first)]
def rem_dupes([], _), do: []
def rem_dupes([same | t], same), do: rem_dupes(t, same)
def rem_dupes([next | t], _last), do: [next | rem_dupes(t, next)]
```
[Kemaskini: Saya telah membuat kod ini lebih idiomatik terima kasih kepada nasihat dari pseudohere di /r/elixir]

Jika anda tidak mengenali Elixir, atau anda tidak begitu faham dengan corak-corak aturcara kefungsian, maka ini mungkin namapak seperti jampi serapah yang menjelekkan.  Anda berfikir "Tengok itu: saya akan habis satu jam untuk selesaikan teka-teki, kemudian selepas itu menulis kod yang tak akan saya fahami lagi selepas ini."  Dan anda adalah salah.  Kepada sesiapa yang berpengalaman dengan Elixir(atau bahasa kefungsian yang lain), kod ini sebenarnya agak mudah untuk dibaca sebab ia menggunakan beberapa corak lazim yang mana-mana pengaturcara Elixir akan ada dalam poket mereka.  Benarkan saya untuk gambarkan:

```elixir
def rem_dupes([]), do: []
```
Ini adalah manuver amat lazim.  Uruskan kes-kes sisi di dalam kepala fungsi berasingan, dan uruskan mereka dahulu.  Tidak perlu runsing tentang mereka lagi.  Adakah ini lebih memeningkan dari cara C?  Mungkin tidak.

```elixir
def rem_dupes([first '|' t]), do: [first '|' rem_dupes(t, first)]
```

Ok, saya mengaku itu agak rumit.  Notasi `[first '|' t]` menyebabkan `first` diikat kepada elemen pertama, dan `t` diikat kepada baki list tersebut.  Apa yang kita buat di sini adalah sama dengan apabila kita membuat penetapan untuk `current_val` dan `current_output_position` di dalam kod C.  Kita menangkap item pertama dan menetapkannya sebagai item "semasa" di dalam panggilan kita kepada fungsi `rem_dupes` versi dua parameter.  Kita sedang membuat persiapan untuk "walk the list"(iaitu memproses setiap item di dalam list secara berturutan).  Menggunakan fungsi berkepala 1-arity(menerima cuma satu argumen) sebagai persediaan untuk kerja berat yang akan dilakukan oleh fungsi 2-arity atau 3-arity adalah corak yang amat lazim dalam Elixir.  Sekarang mungkin nampak pelik, tapi percayalah anda akan lali selepas menggunakannya 100 kali.  Jadi mari lihat kepada fungsi 2-arity, di mana kerja-kerja betul dijalankan:

```elixir
def rem_dupes([],_), do: []
```

Kita telah sampai ke akhir list tersebut, tiada lagi nilai untuk dipulangkan, dan kita tidak hiraukan tentang nilai paling akhir.  Ini akan memeningkan jika anda seorang pengaturcara imperatif,  sebab ia berasa seperti kita sedang melakukan kerja paling akhir dahulu.  Tapi saya tanya, adakah ini berlainan dari memeriksa `i < count` di bahagian atas gelungan-for dan bukannya di bahagian bawah?  Ia cuma satu corak.  Pada satu masa saya perlu berfikir tentangnya, tetapi sekarang ianya jelas apa yang berlaku.

```elixir
def rem_dupes([same '|' t], same), do: rem_dupes(t, same)
```

Sihir apakah ini!?  Di sini kita mengggunakan pemadanan corak untuk melangkau elemen-elemen yang bernilai sama dengan apa yang sedang kita cari(parameter kedua tersebut).  Ini cuma akan padan, dan seterusnya fungsinya dilaksanakan, jika kepala kepada list itu sama nilai dengan parameter kedua.  Pada mulanya aturcara sebegini nampak agak gila-gila tetapi setelah anda menghadamkan corak tersebut, anda akan menulis dan membaca corak itu tanpa berfikir panjang.

```elixir
def rem_duper([next | t], _last), do: [next | rem_dupes(t, next)] 
```

Dan akhir sekali, langkah kita untuk mendapatkan nilai seterusnya.  Perhatika kita melaksanakan "next" sebagai parameter kedua kepada panggilan rekursif itu, dan kita tidak lagi hiraukan tentang nilai "last"(di tandakan dengan _).


Setiap satu baris itu mewakili corak-corak lazim dalam Elixir, dan anda tidak akan menguasai mereka semalaman.  Anda juga tidak menguasai corak-corak yang anda ada sekarang semalaman.  Boleh kata saya mengambil masa sekurang-kurangnya 80-100 jam mengaturcara Elixir, sebelum ia menjadi seratus kali lebih mudah.  Jika ada seseorang berlari masuk ke dalam pejabat hari ini, berpeluh-pelai dan tercungap-cungap, dan memberitahu saya bahawa seseorang diperlukan untuk menulis beberapa kod utuk menghurai sekumpulan fail teks, menjalankan beberapa rumus bisnes kuno, dan mengemaskini satu atau dua pangkalan data, atau dunia akan kiamat, saya pasti saya akan mencapai Elixir dahulu.  Saya yakin bahawa saya akan dapat menyelesaikan apa-apa masalah yang dibawa kepada saya lebih pantas dengan mengguakan ELixir dari peralatan lain.


Jika anda percayakan saya, dan anda mahu mencuba untuk menghadam corak-corak tersebut, ini nasihat saya:

1. Jangan berikan tekanan kepada diri sendiri.  Anggapkan yang anda akan berusaha untuk memahami corak-corak tersebut dahulu, dan jangan bimbangkan berapa lama diperlukan.  Percayakan kawan lama anda Cameron, yang anda akan menguasainya pada akhirnya, dan ianya akan betul-betul berbaloi. 
2. Membuat 100 latihan kecil adalah lebih berguna dari membuat satu latihan besar.  Janga cuba menulis Monopoly versi atas talian atau apa-apa seperti itu dari mula.  Selesaikan jenis-jenis masalah yang anda akan jumpa dalama tahun pertama kelas pengaturcaraan.  Saya senaraikan beberapa sumber yang bagus untuk jenis-jenis masalah ini.
3. Selesaikan jenis masalah yang sama berulang-kali.  Corak-corak mikro ini haruslah menjadi seolah-olah memori otot, dan satu-satunya cara untuk mendapatnya ialah dengan melakukan latihan berulang-ulang kali.  Untuk menerangkan apa yang saya maksudkan, saya telah menulis peyelesaian Elixir kepada masalah tersebut dalam masa kurang dari seminit pagi tadi, dan sekaligus, sebab ia menggunakan corak-corak mikro yang telah berulang-kali saya gunakan.
4. Selesaikan masalah yang sama lagi sekali.  Pergi balik kepada masalah yang telah anda selesaikan dan selesaikannya sekali lagi, dengan kefahaman-kefahaman baru anda.
5. Jangan segan untuk ketepikan satu-satu masalah.  Jika masalah itu tidak menyeronokkan, langkau.  Tiada siapa membayar anda untuk melakukan ini,  dan banyak lagi masalah-masalah lain di luar sana.  Buangkannya dan buat sesuatu yang menyeronokkan.  Mungkin akan kembali semula kepada masalah itu, atau mungkin juga tidak.
6. Minta pertolongan!  [Tim Slack Elixir](https://elixir-slackin.herokuapp.com/), dan [senarai mailing Elixir](https://groups.google.com/forum/#!forum/elixir-lang-talk) adalah tempat paling mesra dalam internet.  Terdapat juga saluran `#beginner` di dalam Slack, dan ramai yang suka menolong di sana sepanjang masa.  Terdapat juga perjumpaan-perjumpaan Elixir di kebanyakan bandar besar.


#### Senarai sumber latihan untuk diselesaikan:

- [Project Euler](https://projecteuler.net/):  Latihan-latihan di sini lebih menjurus kepada matematik, tetapi banyak yang saya belajar dari membuat lebih kurang 50 yang terkehadapan.
- [Exercism](http://exercism.io/): Terdapat saluran Elixir di sini, dan perkara lazim untuk dilakukakan ialah menghantar penyelesaian kepada exercism, kemudian ping saluran "#general" di Slack untuk mencari maklumbalas.  Saya menggalakkan tabiat tersebut.
- [Advent Of Code](http://adventofcode.com/): Ianya amat berguna buat saya, dan amat menyeronokkan.  Terutama 14-15 hari pertama.  Amat disyorkan.
- [Programming Elixir](https://pragprog.com/book/elixir12/programming-elixir-1-2) oleh Dave Thomas.  Ini ialah buku yang hebat, sebagaimana anda jangka dari Dave, dan menawarkan banyak latihan-latihan kecil, bersama dengan forum online di mana penyelesaian dibincangkan.
- [Exercise for Programmers](https://pragprog.com/book/bhwb/exercises-for-programmers) oleh Brian Hogan.  Buku ini mengandungi 57 latihan-latihan kecil.    

