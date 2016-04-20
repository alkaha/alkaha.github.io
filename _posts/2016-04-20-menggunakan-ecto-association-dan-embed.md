---
layout: post
title: Menggunakan Ecto Association dan Embed
---

- Artikel Asal: [Working with Ecto associations and embeds](http://blog.plataformatec.com.br/2015/08/working-with-ecto-associations-and-embeds/)
- Penulis Asal: [José Valim](http://blog.plataformatec.com.br/?author=4)
- Tarikh Terbit: 12 Ogos, 2015

---

Bog post ini bertujuan untuk mendokumen bagaimana untuk menggunakan association di dalam Ecto, meliputi bagaimana untuk membaca, menyimpan, mengemaskini dan memadam association dan embed.  Di penghujung artikel, kami akan memberikan contoh yang lebih rumit yang menggunakan Ecto association untuk membina borang bersarang di dalam Phoenix.

Artikel ini menganggap anda mempunyai pengetahuan asas mengenai Ecto, khususnya bagaimana repositori, skema dan sintaks query berfungsi.  Anda boleh belajar lebih lanjut mengenai mereka di dalam [dokumentasi](http://hexdocs.pm/ecto/) Ecto.

## Association

Association di dalam Ecto digunakan apabila dua sumber data(tables) berbeza dihubungkan menggunakan foreign key.

Contoh klasik ialah situasi "Post memiliki banyak komen".  Mula-mula buat dua table di dalam migration:

```elixir
create table(:posts) do
	add :title, :string
	add :body, :text
	timestamps
end

create table(:comments) do
	add :post_id, references(:posts)
	add :body, :text
	timestamps
end
```

Setiap komen mengandungi satu pemadanan `post_id` yang merujuk kepada satu post `id`.

Sekarang isytiharkan skema:

```elixir
defmodule MyApp.Post do
	use Ecto.Schema

	schema "posts" do
		field :title
		field :body
		has_many :comments, MyApp.Comment
		timestamps
	end
end

defmodule MyApp.Comment do
	use Ecto.Schema

	schema "commentss" do
		field :body
		belongs_to :post, MyApp.Post
		timestamps
	end
end
```

Semua takrifan skema seperti `field`, `has_many` dan lain-lain ditakrifkan di dalam [Ecto.Schema](http://hexdocs.pm/ecto/Ecto.Schema.html).

Sama seperti `has_many/3`, satu skema juga boleh memanggil `has_one/3` apabila table induk hanya mengandungi satu entri anak.  

Perbezaan antara `has_one/3` dan `belongs_to/3` ialah skema yang menggunakan `belongs_to/3` wajib mengandungi foreign key.  Anda boleh mengatakan bahawa skema yang memanggil `has_*` sebagai skema induk dan skema yang memanggil `belongs_to` sebagai skema anak.

Implikasi dari situasi ini boleh dilihat apabila menyimpan dan memadam data dari pangkalan data, di mana apabila menyimpan data, data skema induk akan disimpan dahulu sebelum data skema anak.  Untuk proses memadam, data akan dipadam dari skema anak dahulu sebelum data skema induk dipadamkan.

## Menjalankan query untuk association

Salah satu faedah daripada penakrifan association ialah ia boleh digunakan semasa membuat query.  Sebagai contoh:

```elixir
Repo.all from p in Post,
							preload: [:comments]
``` 

Query ini akan mengambil semua post dari pangkalan data, bersama dengan semua komen yang berkaitan dengan post tersebut.  Contoh diatas akan menjalankan dua query: satu untuk memuatkan semua post dan lagi satu untuk memuatkan semua komen.  Selalunya ini adalah cara yang paling efisien untuk memuatkan association daripada pangkalan data(walaupun dua query berbeza dijalankan) oleh kerana kita cuma mahu menerima dan menghuraikan hasil POSTS + COMMENTS.

Penggunaan `join` juga dibenarkan apabila menjalankan query yang kompleks.  Sebagai contoh, katakan kedua-dua posts dan comments mempunyai undian(votes) dan kita cuma mahukan komen yang mempunyai undian yang lebih tinggi dari undian untuk post itu sendiri:

```Elixir
Repo.all from p in Post
	join: c in assoc(p, :comments),
	where: c.votes > p.votes
	preload: [comments: c]
```

Contoh di atas akan menjalankan satu query sahaja, mencari semua post dan komen berkaitan yang memenuhi syarat tersebut.

## Memanipulasi association

Sebagaimana yang telah disebut di atas, Ecto memainkan peranan yang ketat di dalam hubungan induk-anak.  Sebagai contoh, jika anda mahu membuat satu post bersama komen, anda boleh:

```elixir
Repo.transaction fn ->
	post = Repo.insert!(%Post{title: "Hello", body: "World"})

	comment = Ecto.build_assoc(post, :comments, body: "Excellent")

	Repo.insert!(comment)
end
```

Fungsi `Ecto.build_assoc/3` di atas membina komen menggunakan id yang ditetapkan di dalam struct post.  Ianya sama dengan:

```elixir
%Comment{post_id: post.id, body: "Excellent"}
```

Fungsi `Ecto.build_assoc/3` amat berguna di dalam kontroler Phoenix.  Sebagai contoh, jika kita mahu mengaitkan satu post dengan pengguna yang telah log in, kita boleh buat:

```elixir
Ecto.build_assoc(current_user, :post)
```

Di dalam kontroler lain, kita juga boleh mengaitkan satu komen kepada satu post semasa, dengan:

```elixir
Ecto.build_assoc(post, :comments)
```

Ecto tidak membekalkan fungsi-fungsi seperti `post.comment << comment` yang membenarkan campuran antara persisted data dan non-persisted data.  Mekanisma tunggal untuk mengemaskini kedua-dua post dan komen pada masa yang sama ialah melalui changeset yang akan di lihat apabila membincangkan tentang embed dan association bersarang.

## Memadam association

Apabila mengisytiharkan `has_may/3`, `has_one/3` dan lain-lain fungsi yang sama, anda juga boleh menghantar pilihan `:on_delete` yang menetapkan tindakan untuk dilaksanakan apabila rekod induk dipadamkan.

```elixir
has_many :comments, MyApp.Comment, on_delete: :delete_all
```
Selain daripada di atas, pilihan `:nilify_all` juga disokong, dengan `:nothing` sebagai nilai lalai.  Lihat [dokumentasi has_many/3](http://hexdocs.pm/ecto/Ecto.Schema.html#has_many/3) untuk maklumat lanjut.

## Embed

Selain dari association, Ecto juga menyokong pembenaman(embed) di dalam beberapa pangkalan data.  Dengan pembenaman, satu skema anak akan dibenamkan di dalam skema induk, dan bukannya disimpan di dalam table lain.

Pangkalan data seperti PostgreSQL menggunakan campuran kolum-kolum JSONB( embeds_one/3 ) dan ARRAY untuk menyokong kefungsian ini (kedua-dua JSONB dan ARRAY mendapat sokongan kelas pertama di dalam Ecto).

Penggunaan pembenaman(embeds) adalah lebih kurang sama dengan penggunaan field lain di dalam skema, kecuali apabila masa untk memanipulasi mereka.  Mari kita lihat satu contoh:

```elixir
defmodule MyApp.Permalink do
	use Ecto.Schema

	embedded_schema do
		field :url
		timestamps
	end
end

defmodule MyApp.Post do
	use Ecto.Schema

	schema "posts" do
		field :title
		field :body
		has_many :comments, MyApp.Comment
		embeds_many :permalinks, MyApp.Permalink
		timestamps
	end
end

```

Oleh kerana Ecto perlu mengawasi bagaimana permalink tersebut berubah di dalam post itu, permalink cuma boleh ditambah atau dibuang melalui API changeset.  Mari kita buat satu post yang mengandungi permalink:

```elixir
# Generate a changeset for the post
changeset = Ecto.Changeset.change(post)

# Let's track the new permalinks
changeset = Ecto.Changeset.put_embed(changeset, :permalinks,
	[%Permalink{url: "example.com/thebest"},
	 %Permalink{url: "another.com/mostaccessed"}]
)

# Now let's insert the post with permalinks at once!
post = Repo.insert!(changeset)
```

Sekarang jika anda mahu menukar atau membuang mana-mana permalink, anda boeh menggunakan permalink tersebut sebagai satu collection dan kemudian simpan ia sebagai satu perubahan:

```elixir
# Remove all permalinks from example.com
permalinks = Enum.reject post.permalinks, fn permalink ->
	permalink.url =~ "example.com"
end

# Let's create a new changeset
changeset =
	post
	|> Ecto.Changeset.change
	|> Ecto.Changeset.put_embed(:permalinks, permalinks)

# And update the entry
post = Repo.update!(changeset)
```

Kelebihan menggunakan changeset ialah mereka menyimpan setiap perubahan yang akan dihantar ke pangkalan data dan kita dibenarkan untuk memeriksa perubahan-perubahan tersebut pada bila-bila masa.  Sebagai contoh, jika kita memanggil IO.inspect sebelum menjalankan fungsi `Repo.update!/3`:

```elixir
IO.inspect(changeset.changes.permalinks)

```

Anda akan dapat melihat paparan seperti berikut:

```elixir
[%Ecto.Changeset{action: :delete, changes: %{},
								 model: %Permalink{url: "example.com/thebest"}},
 %Ecto.Changeset{action: :update, changes: %{},
								 model: %Permalink{url: "another.com/mostaccessed"}}]

```

Jika kita telah membuat satu arahan insert untuk menyimpan data, kita juga akan dapat melihat lagi satu changeset yang mempunyai action `:inser`.

Changeset mengandungi satu view yang lengkap tentang apa yang telah berubah, bagaimana mereka berubah dan anad dibenarkan memanipulasi secara lansung.

## Association Bersarang dan pembenaman

Sebagaimana kita gunakan changeset untuk memanipulasi pembenaman, kita juga boleh menggunakan cara yang sama untuk mengubah child association pada yang sama kita memanipulasi induk.

Salah satu faedah dari ciri ini ialah ia boleh digunakan untuk membina borang-borang bersarang(nested forms) di dalam satu aplikasi Phoenix.  

Untuk mengakhiri artikel ini, mari kita lihat satu contoh untuk melihat bagaimana untuk menggunakan apa yang telah kita pelajari setakat ini untuk menggunakan association bersarang di dalam Phoenix.

Pertama sekali, buat satu aplikasi Phoenix.  [Panduan Phoenix](http://phoenixframework.org/) boleh membantu anda bermula jika ini kali pertama anda menggunakan Phoenix.

Contoh yang akan kita bina ialah satu 'to do list', di mana satu satu senarai akan mempunyai banyak item di dalamnya.  Mari kita janakan resource `TodoList`:

```console
mix phoenix.gen.html TodoList todo_lists title
```

Ikuti arahan yang ditunjukkan oleh arahan di atas dan kemudian janakan satu model TodoItem:
```console
mix phoenix.gen.model TodoItem todo_items body:text todo_list_id:references:todo_lists 
```

Buka modul `MyApp.TodoList` di "web/models/todo_list.ex" dan tambahkan takrifan has_many di dalam blok skema:

```elixir
has_many :todo_items, MyApp.TodoItem
```

Seterusnya kita akan tambahkan "todo_items" kepada fungsi changeset `TodoList`:

```elixir
def changeset(todo_list, params \\ :empty) do
	todo_list
	|> cast(params, @required_fields, @optional_fields)
	|> cast_assoc(:todo_items, required: true)
end

```

Oleh kerana kita telah tambahkan `todo_items` sebagai medan wajib, borang kita telah sedia untuk dihantarkan.  Sekarang kita akan ubah templat kita untuk menghantar item-item todo juga.  Buka `web/templates/todo_list/form.html.eex` dan tambahkan kod di bawah di antar input title dan butang submit:

```elixir
<%= inputs_for f, :todo_items, fn i -> %>
	<div class="form-group">
		<%= label i, :body, "Task ##{i.index + 1}", class: "control-label" %>
		<%= text_input i, :body, class: "form-control" %>
		<%= if message = i.errors[:body] do %>
			<span class="help-block"><%= message %></span>
		<% end %>
	</div>
<% end %>
```

Fungsi `inputs_for/4` datang dari [Phoenix.HTML.Form](http://hexdocs.pm/phoenix_html/Phoenix.HTML.Form.html#inputs_for/4) dan ia mengupayakan kita untuk menjana medan-medan untuk association dan embed, menghasilkan satu struct borang (diwakili oleh pembolehubah `i` di dalam contoh di atas) untuk kita gunakan.  Di dalam fungsi `inputs_for/4`, kita menjana satu input teks untuk setiap item.

Setelah kita membuat perubahan tersebut, langkah terakhir ialah untuk mengubah action `new` di dalam controller untuk supaya ia mengandungi dua item todo yang kosong secara lalai di dalam senarai todo:

```elixir
changeset = TodoList.changeset(%TodoList{todo_items: [%MyApp.TodoItem{}, %MyApp.TodoItem{}]})
```

Sekarang apabila kita capai “http://localhost:4000/todo_lists”  kita akan dapat melihat satu senarai todo dengan kedua-dua todo item!  Bagaimanapun, jika kita cuba ubah senarai todo yang baru dibuat itu, kita akan mendapat ralat berikut:

```console
attempting to cast or change association :todo_items for MyApp.TodoList that was not loaded.
Please preload your associations before casting or changing the model.
```

Sebagaimana yang disebut oleh ralat itu, kita perlu untuk preload item-item todo kepada action `edit` dan `update` di dalam `MyApp.TodoListController`.

Buka controller tersebut dan ubahkan baris berikut untuk kedua-dua action tersebut:

```elixir
todo_list = Repo.get!(TodoList, id)
```

kepada

```elixir
todo_list = Repo.get!(TodoList, id) |> Repo.preload(:todo_items)
```

Sekarang sepatutnya kita sudah boleh mengemaskini item-item todo di samping senarai todo itu sendiri.

Kedua-dua operasi insert dan update adalah dikuasakan oleh changeset,  sebagaimana yang dapat kita lihat di dalam action controller:

```elixir
changeset = TodoList.changeset(todo_list, todo_list_params)
```

Kesemua faedah changeset yang telah kita bincangkan sebelum ini masih digunakan di sini.  Dengan memeriksa changeset sebelum memanggil `Repo.insert` atau `Repo.update`,  kita boleh melihat semua perubahan yang akan berlaku kepada pangkalan data.

Selain dari itu,  proses validasi di dalam changeset adalah eksplisit.  Oleh sebab kita telah tambahkan `todo_items` sebagai medan wajib,  setiap kali kita panggil `MyApp.TodoList.changeset/2`, fungsi `MyApp.TodoItem.changeset/2` juga akan dipanggil setiap kali satu item todo di hantar melalui borang tersebut.  Changeset yang dihasilkan oleh setiap todo item akan disimpan di dalam changeset senarai induk todo.

Untuk menambah lagi kefahaman kita terhadap changeset, mari tambahkan beberapa validasi ke atas item-item todo dan juga benarkan mereka untuk dipadamkan.

## Memadam item-item todo

Buka fail “web/models/todo_item.ex” dan tambahkan satu medan maya bernama `:delete` ke dalam skema:

```elixir
field :delete, :boolean, virtual: true
```

Sebagaimana yang kita tahu fungsi `MyApp.TodoItem.changeset/2`  adalah fungsi yang dipanggil apabila memanipulasi item-item todo melalui senarai todo.  Jadi kita ubahkan kepada berikut:

```elixir
@required_fields ~w(body)
@optional_fields ~w(delete) # 1. Make delete an optional field

def changeset(todo_item, params \\ :empty) do
  todo_item
  |> cast(params, @required_fields, @optional_fields)
  |> validate_length(:body, min: 3)
  |> mark_for_deletion() # 2. Call mark for deletion
end

defp mark_for_deletion(changeset) do
  # If delete was set and it is true, let's change the action
  if get_change(changeset, :delete) do
    %{changeset | action: :delete}
  else
    changeset
  end
end
```

Sekarang kita telah tambahkan satu panggilan kepada `validate_length` dan juga kepada satu fungsi terlindung yang memeriksa jika medan `:delete` berubah, dan jika benar, kita tandakan action changeset kepada `:delete`.

Fungsi-fungsi `cast`, `validate_length`, `get_change` dan lain-lain adalah sebahagian dari [module Ecto.Changeset](http://hexdocs.pm/ecto/Ecto.Changeset.html) yang dimport secara otomatik ke dalam model-model Ecto.

Sekarang ubahkan view kita supaya memilik medan delete.  Tambahkan berikut ke dalam mana-mana bahagian di dalam panggilan `inputs_for/4` di dalam fail “web/templates/todo_list/form.html.eex”:

```elixir
<%= if i.model.id do %>
  <span class="pull-right">
    <%= label i, :delete, "Delete?", class: "control-label" %>
    <%= checkbox i, :delete %>
  </span>
<% end %>
```

Itu sahaja.  Sekarang item-item todo kita sedia untuk membuat validasi dan juga membnarkan pemadaman di laman update.

Walaupun memanggil `MyApp.TodoItem.changeset/2` adalah merupakan kaedah lalai,  kita masih boleh mengubahsuai fungsi tersebut untuk dipanggil apabila membuat casting todo item dari dalam changeset todo list melalui arahan `:with`:

```elixir
|> cast_assoc(:todo_items, required: true, with: &custom_changeset/2)
```

Oleh itu jika satu association mempunyai peraturan validasi yang berbeza bergantung kepada jika ia dihantar melalui association bersarang atau diuruskan secara lansung,  kita boleh asingkan business rule tersebut dengan menyediakan dua fungsi changeset yang berbeza.  Dan juga sebab kita menggunakan fungsi-fungsi di dalam seluruh operasi ini, mereka adalah senang untuk dikarang dan diuji.

##  Ringkasan

Di dalam artikel ini kita telah belajar asas yang diperlukan untuk menggunakan association dan embed,  sehingga kepada contoh yang agak rumit menggunakan association bersarang.  Jika anda mahu membuat lebih banyak pengubahsuaian, baca dokumentasi untk mentakrifkan association/embed di dalam [Ecto.Schema](http://hexdocs.pm/ecto/Ecto.Schema.html) atau bagaimana untuk memanipulasi changeset melalui [Ecto.Changeset](http://hexdocs.pm/ecto/Ecto.Changeset.html).

Maklumat lanjut mengenai view boleh di dapati di dalam projek `Phoenix.HTML`, khususnya di bawah [Phoenix.HTML.Form](http://hexdocs.pm/phoenix_html/Phoenix.HTML.Form.html), di mana fungsi `inputs_for/4` ditakrifkan.
