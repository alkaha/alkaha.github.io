---
layout: post
title: Membina Elixir Blog Dalam 15 Minit Menggunakan Phoenix - Langkah Demi Langkah
---

- Dokumen asal:  [Elixir blog in 15 minutes using Phoenix framework - Step by Step](http://codetunes.com/2015/phoenix-blog/) 
- Penulis asal: [Jakub Cieslar](http://jcieslar.pl/) 
- Ringkasan: Artikel ini memperkenalkan penggunaan framework Phoenix utuk membina satu aplikasi blog yang ringkas. 

---

Saya jangkakan masih ramai yang mengingati 'blog post' membina blog sendiri dalam masa 15 minit menggunakan Ruby On Rails.  Membina blog sendiri menggunakan Ruby On Rails adalah semudah menulis aplikasi 'Hello World' di dalam lain-lain bahasa.  Sekarang ini, semakin banyak artikel mengenai Elixir dan nampaknya semakin ramai pengguna Ruby yang 'jatuh hati' dengan Elixir.  Oleh sebab itu, saya mengambil cabaran untuk melihat samada mudah (atau susah ;)) untuk membina blog melalui Elixir dengan menggunakan Phoenix, iaitu satu 'web framework' Elixir.

Saya juga cadangkan anda meneliti [tutorial Elixir ini]() dan [tutorial Phoenix ini]().  Dan untuk mereka yang masih suka membaca, saya cadangkan buku 'Programming Elixir' oleh Dave Thomas.

Tujuan utama artikel ini adalah untuk menarik perhatian anda kepada Elixir dan Phoenix secara keseluruhannya.  Saya tidak akan memberikan penjelasan mendalam bagaimana keduanya berfungsi.  Sebaliknya saya cuma mahu menunjukkan betapa mudahnya untuk kita menulis satu aplikasi yang semudah blog menggunakan Elixir.

## Persediaan

Pertama sekali, kita perlu memasang Elixir: [http://elixir-lang.org/install.html](http://elixir-lang.org/install.html]).  Seterusnya, pasang 'Hex package manager' dan Phoenix:
{% highlight js %}

{% endhighlight %}

Langkah-langkah terperinci dijelaskan di [sini]().

## Langkah 1

Mulakan projek baru bertajuk 'blog_phoenix':
{% highlight js %} 
$ mix phoenix.new blog_phoenix
{% endhighlight %} 

Kita akan dapat melihat fail-fail berikut dihasilkan:
{% highlight js %} 
* creating blog_phoenix/config/config.exs
* creating blog_phoenix/config/dev.exs
* creating blog_phoenix/config/prod.exs
* creating blog_phoenix/config/prod.secret.exs
* creating blog_phoenix/config/test.exs
* creating blog_phoenix/lib/blog_phoenix.ex
* creating blog_phoenix/lib/blog_phoenix/endpoint.ex
* creating blog_phoenix/test/controllers/page_controller_test.exs
* creating blog_phoenix/test/views/error_view_test.exs
* creating blog_phoenix/test/views/page_view_test.exs
* creating blog_phoenix/test/views/layout_view_test.exs
* creating blog_phoenix/test/support/conn_case.ex
* creating blog_phoenix/test/support/channel_case.ex
* creating blog_phoenix/test/test_helper.exs
* creating blog_phoenix/web/channels/user_socket.ex
* creating blog_phoenix/web/controllers/page_controller.ex
* creating blog_phoenix/web/templates/layout/app.html.eex
* creating blog_phoenix/web/templates/page/index.html.eex
* creating blog_phoenix/web/views/error_view.ex
* creating blog_phoenix/web/views/layout_view.ex
* creating blog_phoenix/web/views/page_view.ex
* creating blog_phoenix/web/router.ex
* creating blog_phoenix/web/web.ex
* creating blog_phoenix/mix.exs
* creating blog_phoenix/README.md
* creating blog_phoenix/lib/blog_phoenix/repo.ex
* creating blog_phoenix/test/support/model_case.ex
* creating blog_phoenix/priv/repo/seeds.exs
* creating blog_phoenix/.gitignore
* creating blog_phoenix/brunch-config.js
* creating blog_phoenix/package.json
* creating blog_phoenix/web/static/css/app.scss
* creating blog_phoenix/web/static/js/app.js
* creating blog_phoenix/web/static/assets/robots.txt
* creating blog_phoenix/web/static/vendor/phoenix.js
* creating blog_phoenix/web/static/assets/images/phoenix.png
* creating blog_phoenix/web/static/assets/images/favicon.ico
{% endhighlight %}

Kemudian kita perlu memasang 'dependencies':
{% highlight js %} 
$ cd blog_phoenix
$ mix deps.get
{% endhighlight %}

Selepas 'compiling' kita akan mendapat aplikasi yang sedia untuk digunakan di: `http://localhost:4000`.

## Langkah 2

Sekarang kita sudah bersedia untuk menulis fungsi-fungsi teras pada aplikasi ini.  Kita memerlukan kemudahan untuk membuat kerja-kerja `CRUD` untuk setiap `post` dan juga kemudahan untuk membuat `comment` kepada setiap `post` (ianya satu aplikasi blog yang ringkas).  Untuk mencapai tujuan ini, Phoenix menyediakan 4 jenis `generator`:
{% highlight ruby %} 
$ mix phoenix.gen.html →  yang menghasilkan 'model', 'view', 'controllers', 'repository', 'templates', 'tests'
$ mix phoenix.gen.channel →  yang menghasilkan 'channel' dan 'tests'
$ mix phoenix.gen.json →  untuk kegunaan API, menghasilkan 'model', 'view', 'controllers', 'repository', 'tests'
$ mix phoenix.gen.model →  yang menghasilkan 'model' dan 'repository'
{% endhighlight %}

Kita gunakan 'generator' pertama utuk meghasilkan kesemua 'resource' dan 'action' - lebih kurang sama dengan kegunaan 'generator' Ruby on Rails.  Kita perlu untuk menggunakan nama dalam betuk 'singular' dan 'plural', dan seterusnya nama 'field' bersama dengan 'type'.
{% highlight js %} 
$ mix phoenix.gen.html Post posts title:string body:text
{% endhighlight %}

Sekarang kemudahan CRUD untuk entiti `Post` sudah tersedia.

Berikut adalah senarai fail yang dihasilkan:
{% highlight ruby %} 
* creating priv/repo/migrations/20150730233126_create_post.exs
* creating web/models/post.ex
* creating test/models/post_test.exs
* creating web/controllers/post_controller.exs
* creating web/templates/post/edit.html.eex
* creating web/templates/post/form.html.eex
* creating web/templates/post/index.html.eex
* creating web/templates/post/new.html.eex
* creating web/templates/post/show.html.eex
* creating web/views/post_view.ex
* creating test/controllers/post_controller_test.exs
{% endhighlight %}

Sebelum kita 'refresh' browser, kita perlu menambah 'endpoint' baru di dalam fail `web/router.ex`.
{% highlight ruby %} 
resources "/posts", PostController
{% endhighlight %}

Untuk melihat senarai `routing` kita boleh gunakan:
{% highlight ruby %}
 $ mix phoenix.routes
{% endhighlight %}

yang juga agak sama dengan yang di dalam dunia Rails.  Phoenix menggunaka Ecto secara 'default' untuk berkomunikasi dan berinteraksi dengan pangkala data.  Ecto menyediakan 'adapter' untuk PostgreSQL, MySQL dan SQLite (jumlah pangkalan data yang disokong semakin bertambah).  Kita boleh dapatkan penjelasan yang terperinci untuk 'library' [Ecto]() di bawah dokumentasi Phoenix di [Github]().

Ecto memberikan kita kemudahan untuk menghasilkan 'table' `Post` di dalam pangkalan data dengan menjalankan skrip 'migration':
{% highlight ruby %}
$ mix ecto.migrate 
{% endhighlight %}

Senarai fail yang dihasilkan oleh skrip ini berada di `priv/repo/migrations/`.

Jika kita jalankan kod di atas, kita akan mendapat ralat: kerana kita masih belum membuat pangkalan data, jadi untuk itu kita perlu jalankan:
{% highlight ruby %}
$ mix ecto.create
$ mix ecto.migrate
{% endhighlight %}

Anda mungkin nampak yang 'command' `mix ecto.something` adalah lebih kurang sama dengan `rake db:something` di dalam Rails.

Melalui `http://localhost:4000/post` kita akan dapat melihat bagaimana kemudahan CRUD untuk entiti `Post` telah pun dijanakan.  Sila uji kemudahan tersebut.

## Langkah 3

Langkah seterusnya di dalam aplikasi blog kita adalah untuk menyediakan kemudahan 'comment', iaitu kemudahan untuk melihat senarai komen dan menambah komen baru.  Kita akan membenarkan banyak komen untuk setiap `post`.  Kita akan gunakan 'generator' yang akan menghasilkan 'model' dan 'migration' untuk entiti `Comment`:
 
{% highlight ruby %}
$ mix phoenix.gen.model Comment comments name:string content:text post_id:references:posts
{% endhighlight %}

Untuk membuat hubungkait antara entiti `Post` dan entiti `Comment`, kita menggunakan `post:references` - sama seperti di dalam Rails.  Kita juga perlu untuk menambah `has_many: comments, BlogPhoenix.Comment` ke dalam fail `web/models/post.ex` seperti berikut:
{% highlight ruby %}
defmodule BlogPhoenix.Post do
  use BlogPhoenix.Web, :model

  schema "posts" do
    field :title, :string
    field :body, :string

    has_many :comments, BlogPhoenix.Comment

    timestamps
  end
end
{% endhighlight %}

Kemudian kita jalankan 'migration' seperti:
{% highlight ruby %}
$ mix ecto.migrate
{% endhighlight %}

Seterusnya kita perle menambah fungsi `add_comment` di dalam fail `web/router.ex`:
{% highlight ruby %}
resources "/posts", PostController do
  post "/comment", PostController, :add_comment
end
{% endhighlight %}

Kita baru menambah 'resource' untuk `comment` di dalam 'resource' untuk `post`. Kita boleh melihat 'routing table' yang baru seperti berikut:
{% highlight ruby %}

{% endhighlight %}

Seterusnya kita perlu membuat perubahan pada `PostController`(iaitu fail `web/controllers/post_controller.ex`).  Untuk mendapat akses kepada 'model' `Comment`, kita perlu menambah:
{% highlight ruby %}
alias BlogPhoenix.Comment
{% endhighlight %}

Baca mengenai 'alias' di [sini]().

Seterusnya tambah 'scrub params' pada bahagian atas 'controller'.  'Scrub params' adalah sama dengan 'strong parameters' di dalam Rails.  Baca mengenai 'scrub params' di [sini]().
{% highlight ruby %}
plug :scrub_params, "comment" when action in [:add_comment]
{% endhighlight %}

dan 'function' `add_comment`:
{% highlight ruby %}
def add_comment(conn, %{"comment" => comment_params, "post_id" => post_id}) do
  changeset = Comment.changeset(%Comment{}, Map.put(comment_params, "post_id", post_id))
  post = Post |> Repo.get(post_id) |> Repo.preload([:comments])

  if changeset.valid? do
    Repo.insert(changeset)

    conn
    |> put_flash(:info, "Comment added.")
    |> redirect(to: post_path(conn, :show, post))
  else
    render(conn, "show.html", post: post, changeset: changeset)
  end
end
{% endhighlight %}

Fungsi `changeset` yang digunakan di dalam kod di atas boleh di dapati di dalam fail `web/models/comment.ex`.  Baca mengenai Ecto `changeset` di [sini]().

Seterusnya kita perlu mengubah 'function' `show` seperti berikut:
{% highlight ruby %}
def show(conn, %{"id" => id}) do
  post = Repo.get(Post, id) |> Repo.preload([:comments])
  changeset = Comment.changeset(%Comment{})
  render(conn, "show.html", post: post, changeset: changeset)
end
{% endhighlight %}
untuk membenarkan kita meletakkan 'comment form' di dalam 'view' untuk `post`.

Seterusnya kita perlu membuat 'comment form' tersebut di dalam 'template', iaitu fail `web/templates/post/comment_form.html.eex`:
{% highlight ruby %}
<%= form_for @changeset, @action, fn f -> %>
  <%= if f.errors != [] do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below:</p>
      <ul>
        <%= for {attr, message} <- f.errors do %>
          <li><%= humanize(attr) %> <%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="form-group">
    <label>Name</label>
    <%= text_input f, :name, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Content</label>
    <%= textarea f, :content, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Add comment", class: "btn btn-primary" %>
  </div>
<% end %>
{% endhighlight %}

dan 'render' 'template' tersebut di dalam fail `web/templates/post/show.html.eex`.
{% highlight ruby %}
<%= render "comment_form.html", post: @post, changeset: @changeset,
action: post_path(@conn, :add_comment, @post) %>
{% endhighlight %}

## Langkah 4

Sekarang kita boleh menambah komen kepada 'post' kita, tetapi masih belum boleh melihat hasil dari operasi tersebut.  Untuk melihat semua komen yang tambah kepada 'post', kita perlu membuat satu 'partial' baru, iaitu `web/templates/post/comments.html.eex` dan di dalamnya kita akan 'iterate' kesemua komen kepada post tersebut dan memperlihatkan 'author' dan kandungan setiap komen.
{% highlight ruby %}
<h3> Comments: </h3>
<table class="table">
  <thead>
    <tr>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
<%= for comment <- @post.comments do %>
    <tr>
      <td><%= comment.name %></td>
      <td><%= comment.content %></td>
    </tr>
<% end %>
  </tbody>
</table>
{% endhighlight %}

Seterusnya kita perlu 'render' 'template' tersebut di dalam `web/templates/post/show.html.eex`.
{% highlight ruby %}
<%= render "comments.html", post: @post %>
{% endhighlight %}


## Langkah 5

Langkah terakhir adalah kita mahu menunjukkan jumlah komen kepada sesatu 'post' di dalam senarai blog post kita.  Kita perlu menyediakan satu 'query' di mana kita boleh menjumlahkan komen-komen tersebut.  Ianya boleh dilakukan dengan membuat penambahan berikut kepada fail `web/models/post.ex`:
{% highlight ruby %}
defmodule BlogPhoenix.Post do
  use BlogPhoenix.Web, :model
  import Ecto.Query

  ...

  def count_comments(query) do
    from p in query,
      group_by: p.id,
      left_join: c in assoc(p, :comments),
      select: {p, count(c.id)}
  end
end
{% endhighlight %}

Kita akan gunakan 'function' `count_comments` di atas di dalam 'function' index di dalam fail `web/controllers/post_controller.ex`:
{% highlight ruby %}
def index(conn, _params) do
  posts = Post
  |> Post.count_comments
  |> Repo.all
  render(conn, "index.html", posts: posts)
end
{% endhighlight %}

Seterusnya kita perlu membuat perubahan kepada fail `web/templates/post/index.html.eex` kepada berikut:
{% highlight ruby %}
<h2>Listing posts</h2>
<table class="table">
  <thead>
    <tr>
      <th>Title</th>
      <th>Comments</th>

      <th></th>
    </tr>
  </thead>
  <tbody>
<%= for {post, count} <- @posts do %>
    <tr>
      <td><%= post.title %></td>
      <td><%= count %></td>

      <td class="text-right">
        <%= link "Show", to: post_path(@conn, :show, post), class: "btn btn-default btn-xs" %>
        <%= link "Edit", to: post_path(@conn, :edit, post), class: "btn btn-default btn-xs" %>
        <%= link "Delete", to: post_path(@conn, :delete, post), method: :delete, class: "btn btn-danger btn-xs" %>
      </td>
    </tr>
<% end %>
  </tbody>
</table>
{% endhighlight %}

Kita boleh melihat bagaimana blog post kita berfungsi di:
`http://localhost:4000/posts`


## Ringkasan:

'Source code' untuk aplikasi ini boleh di dapati di [Github]().  Aplikasi blog ini ada amat ringkas dan ianya cuma satu percubaan untuk menunjukkan bagaimana mudahnya untuk bermain dengan Elixir dan Phoenix jika kita mempunyai latarbelakang Ruby On Rails.  Sepertimana yang dapat kita lihat, ianya semudah jika ianya dibina menggunakan Rails!  Kita dapat melihat beberapa kesamaan antara Elixir/Phoenix dan Ruby On Rails dari segi 'convention' yang digunakan.  Framework ini menyediakan banyak konsep yang telah dikenali seperti 'models', 'routing', 'controllers' dan 'form helpers', dan juga beberapa konsep baru seperti 'repositories' dan 'changesets(channels)'.  Dengan itu kita merasa selesa untuk menulis kod, tetapi kita perlu ingat bahawa kita tidak lagi menggunakan kaedah 'Object Oriented Programming'.  Oleh itu kita perlu mengubah minda kita kepada mod 'Functional Programming' - dan ianya mengujakan!

Saya berharap ini akan menggalakkan anda untuk melihat lebih dekat akan Elixir dan Phoenix.  Siapa tahu, mungkin Elixir on Phoenix menjadi standard untuk generasi baru web?  
