---
layout: post
title: Membina Test Driven API Menggunakan Elixir dan Phoenix
---

- Artikel asal: [Test-Driven APIs with Phoenix and Elixir](https://semaphoreci.com/community/tutorials/test-driven-apis-with-phoenix-and-elixir)
- Penulis Asal: [Jader Correa](https://semaphoreci.com/community/authors/jadercorrea)
- Ringkasan: Artikel ini menunjukkan langkah-langkah untuk membina aplikasi API menggunakan kaedah Test Driven Development(TDD).

---

## Pengenalan

Bermula dengan teknologi baru boleh menjadi agak sukar.  Selalunya akan mengambil masa untuk memahami komponen-komponen teknologi tersebut dan cara mereka bekerjasama.  Di dalam tutorial ini, kita akan membina satu API, melalui kaedah Test Driven Development(TDD) dalam proses pembinaan.  Anda akan belajar bagaimana maklumbalas dari ujian-ujian membantu kita untuk menyelamatkan banyak masa.  Ianya dilakukan dengan memberikan kita tanda-tanda melalui kegagalan ujian dan memberikan kita pemahaman bagaimana semua komponen bekerja.

## Keperluan

Pertama sekali, pastikan kita ada semua komponen yang diperlukan:

- [Erlang 18.1](http://www.erlang.org/download.html)
- [Elixir 1.1.1](http://elixir-lang.org/install.html)
- [Phoenix 1.0.4](http://www.phoenixframework.org/docs/installation)

Pastikan kita menggunakan versi Elixir yang sama:

```console
$ elixir --version
Elixir 1.1.1
```

## Permulaan

Kita akan bermula dengan memikirkan tentang apa yang kita mahukan dari implementasi ini.  Jika kita terus menggunakan aplikasi penjanaan(generators), ia mungkin berfungsi, tetapi kita akan mempunyai banyak kod yang tidak diperlukan, dan tiada kefahaman tentang lapisan-lapisan kod tersebut.  Oleh sebab kita mahu membangunkan satu API, kita tidak perlukan sebarang HTML dan stylesheet.  Ianya satu tempat yang bagus untuk bermula:

```console
$ mix phoenix.new watchlist --no-html --bo-brunch
```

Pastikan anda menjawab `Y` untuk memasang dependency:

```console
Fetch and install dependencies? [Yn] Y
```

Kita tidak perlukan pangkalan masa buat masa ini, tetapi kita akan bina satu pangkalan data, kerana kita menggunakannya pada peringkat seterusnya.  Kita akan gunakan pangkalan data asas Phoenix, iaitu PostgreSQL.

```console
$ mix ecto.create
```

Ini akan mengkompil projek dan membuat ujian hubungan dengan pangkalan data kita.  Kegagalan `ecto.create` akan dipaparkan seperti berikut:

```console
** (Mix) The database for Watchlist.Repo could not be created, reason given: psql: could not connect to server: Connection refused
```

Kadangkala apabila sistem pengendalian(OS) kita runtuh(crash), satu fail 'lock' untuk Postgres akan tersimpan di dalam sistem fail, menyebabkan Postgres menyangka bahawa satu hubungan telah sedia ada:

```console
$ rm /usr/local/var/postgres/postmater.id
```

Mulakan( atau mulakan semula) pelayan Postgres anda, dan sedia untuk meneruskan:

```console
$ mix ecto.create
The database for Watchlist.Repo has been created.
```

Sekarang semuanya nampak bagus.  Kita boleh mula memikirkan tentang ciri pertama aplikasi.

## Membina satu ciri: Menyenaraikan Wayang

Tiba masanya untuk memikirkan tentang ciri tersebut.  Kita sepatutnya mengelakkan dari berurusan dengan perincian perlaksanaan sebanyak mungkin.

### Menambah satu API Endpoint

Apa yang kita tahu setakat ini? Kita akan memulangkan satu sambutan JSON, bermakna kita memerlukan satu pustaka(library) JSON.  Kita perlukan pustaka `Poison`.(Pustaka ini telah siap dipasangdalam dalam versi Phoenix terkini).

Sekarang kita akan membina satu ujian integrasi.  Buat satu direktori bernama `test/integration` dan satu fail bernama `listing_movies_test.exs` di dalam direktori tersebut.

Menyenaraikan wayang memerlukan satu permohonan `get` kepada satu sumber data wayang.  Kita perlukan satu permohonan `HTTP GET` kepada URI `/movies` untuk memulangkan beberapa kandungan dan satu kod status `200`.

Kita akan bermula dengan:

```elixir
defmodule ListingMoviesIntegrationTest do
  use ExUnit.Case, async: true

  test 'listing movies' do
    response = conn(:get, '/movies')
    assert response.status == 200
  end
end
```

Ini cara bagaimana ia berfungsi:  kita akan menambahkan ujian-ujian sebegini dan jalankan mereka.  Selepas mejalankan ujian tersebut, kita akan membuat perubahan setiap kali ia gagal sehingga kita mempunya kod secukupnya untuk menjayakannya.  Kita akan guna `mix test test/integration/listing_movies_test.exs` untuk menjalankan ujian tersebut.  Masa ini, ujian kita akan memulangkan ralat berikut:

```console
** (CompileError)
test/integration/listing_movies_test.exs:5: function conn/2 undefined
```  

Kita perlukan `Plug`.  Tambahakan berikut kepada fail `test/integration/listing_movies_test.exs`:

```elixir
use Plug.Test
```

Apabila menjalankan semula ujian, kita akan mendapat paparan berikut:

```console
 1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:5
     ** (FunctionClauseError) no function clause matching in URI.parse/1
     stacktrace:
       (elixir) lib/uri.ex:292: URI.parse('/movies')
       (plug) lib/plug/adapters/test/conn.ex:10: Plug.Adapters.Test.Conn.conn/4
       test/integration/listing_movies_test.exs:6

```
Masalah di sini ialah penggunaan single quotes('').  Elixir menggunakan double quotes("") untuk string.  Single quotes('') adalah untuk senarai aksara.  Ubahkan kepada berikut:

```elixir
response = conn(:get, "/movies")
```

```console
  1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:5
     Assertion with == failed
     code: response.status() == 200
     lhs:  nil
     rhs:  200
     stacktrace:
       test/integration/listing_movies_test.exs:7

```

Kita tidak mempunyai kod status kerana kita membuat satu hubungan, tetapi kita tidak membuat sebarang panggilan kepada endpoint dengannya.  Kita tambahkan satu.  Kita perlukan satu 'router', hubungkannya, dan dapatkan sambutan(response):

```elixir
defmodule ListingMoviesIntegrationTest do
  use ExUnit.Case, async: true
  use Plug.Test
  alias Watchlist.Router

  @opts Router.init([])
  test 'listing movies' do
    conn = conn(:get, "/movies")
    response = Router.call(conn, @opts)
    assert response.status == 200
  end
end
```

```console
   1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:7
     ** (Phoenix.Router.NoRouteError) no route found for GET /movies (Watchlist.Router)
     stacktrace:
       (watchlist) web/router.ex:1: Watchlist.Router.match/4
       (watchlist) web/router.ex:1: Watchlist.Router.do_call/2
       test/integration/listing_movies_test.exs:9
```

Ujian gagal lagi kerana kita tidak mempunyai 'route'.  Tambahkan 'route' ke dalam fail `web/router.ex`:

```elixir
scope "/", Watchlist do
  pipe_through :api

  get "/movies", MovieController, :index
end
```

Kita jalankan semula ujian walaupun kita tahu ia akan gagal.  Dalam kes ini ia akan memaparkan:

```console
1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:7
     ** (Plug.Conn.WrapperError) ** (UndefinedFunctionError) undefined function: Watchlist.MovieController.init/1 (module Watchlist.MovieController is not available)
     stacktrace:
       Watchlist.MovieController.init(:index)
       (watchlist) web/router.ex:1: anonymous fn/1 in Watchlist.Router.match/4
       (watchlist) lib/phoenix/router.ex:255: Watchlist.Router.dispatch/2
       (watchlist) web/router.ex:1: Watchlist.Router.do_call/2
       test/integration/listing_movies_test.exs:9

```

Jadi kita perlukan satu 'controller'.  Buat satu fail `web/controllers/movie_controller.ex`:

```elixir
defmodule Watchlist.MovieController do
  use Watchlist.Web, :controller

  def index(conn, _params) do
    render conn, movies: []
  end
end
```

```console
  1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:7
     ** (Plug.Conn.WrapperError) ** (UndefinedFunctionError) undefined function: Watchlist.MovieView.render/2 (module Watchlist.MovieView is not available)

```

Kita juga perlukan 'view'.  Buat fail `web/view/movie_view.ex`:

```elixir
defmodule Watchlist.MovieView do
  use Watchlist.Web, :view

  def render("index.json", %{movies: movies}) do
    movies
  end
end
```

Jalankan semula ujian:

```console
Compiled web/views/movie_view.ex
Generated watchlist app
.

Finished in 0.2 seconds (0.2s on load, 0.01s on tests)
1 test, 0 failures

```

Ujian pertama kita berjaya.  Tambahkan lagi satu ujian untuk memastikan kita di jalan yang betul:

```elixir
assert response.resp_body == "[]"
```

Ujian tersebut berjaya.  Sekarang, cuba dapatkan sumber tersebut dari 'bash', jalankan pelayan dengan `mix phoenix.server`, dan gunakan curl untuk mendapatkan senarai movie:

```console
$ curl localhost:400/movies
[]
```

### Menambahkan Model

Ia berjaya - kita berjaya mendapat satu senarai kosong.  Sekarang, kita perlu tambahkan satu model untuk movie untuk memegang senarai kita.  Lagi sekali, kita akan menggunakan TDD untuk meneraju bagaimana model kita akan dibentuk.  Kita berikan model itu satu medan 'name' dan satu medan 'rating', dan kemudian harapkan ianya dipulangkan:

```elixir
defmodule ListingMoviesIntegrationTest do
  use ExUnit.Case, async: true
  use Plug.Test
  alias Watchlist.Router

  @opts Router.init([])
  test 'listing movies' do
    movie = %Movie{name: "Back to the future", rating: 5}
            |> Repo.insert!

    conn = conn(:get, "/movies")
    response = Router.call(conn, @opts)

    assert response.status == 200
    assert response.resp_body == movie
  end
end
```

```console
** (CompileError) test/integration/listing_movies_test.exs:8: Movie.__struct__/0 is undefined, cannot expand struct Movie
    (elixir) src/elixir_map.erl:58: :elixir_map.translate_struct/4

```

Struktur tersebut masih belum wujud.  Kita gunakan penjana untuk membinanya:

```console
$ mix phoenix.gen.model Movie movies name:string rating: integer
* creating priv/repo/migrations/20151204182719_create_movie.exs
* creating web/models/movie.ex
* creating test/models/movie_test.exs
``` 

Jalankan `mix ecto.migrate` dan kemudian lihat fail `test/model/moview_test.exs` yang dijanakan untuk kita:

```elixir
defmodule Watchlist.MovieTest do
  use Watchlist.ModelCase

  alias Watchlist.Movie

  @valid_attrs %{name: "some content", rating: 42}
  @invalid_attrs %{}

  test "changeset with valid attributes" do
    changeset = Movie.changeset(%Movie{}, @valid_attrs)
    assert changeset.valid?
  end

  test "changeset with invalid attributes" do
    changeset = Movie.changeset(%Movie{}, @invalid_attrs)
    refute changeset.valid?
  end
end
```

Jalankan ujian berikut:

```console
$ mix test test/models/movie_test.exs
..

Finished in 0.2 seconds (0.2s on load, 0.00s on tests)
2 tests, 0 failures

```

Apabila kita menjalankan ujian integrasi semula, kita akan mendapat ralat berikut:

```console
** (CompileError) test/integration/listing_movies_test.exs:8: Movie.__struct__/0 is undefined, cannot expand struct Movie
    (elixir) src/elixir_map.erl:58: :elixir_map.translate_struct/4

```
Kita telah membina struktur data, tetapi belum masukkannya ke dalam ujian.  Tambahkan arahan berikut:

```elixir
alias Watchlist.Movie`
```

Jalankan ujian sekali lagi:

```console
 1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:8
     ** (UndefinedFunctionError) undefined function: Repo.insert!/1 (module Repo is not available)
     stacktrace:
       Repo.insert!(%Watchlist.Movie{__meta__: #Ecto.Schema.Metadata<:built>, id: nil, inserted_at: nil, name: "Back to the future", rating: 5, updated_at: nil})
       test/integration/listing_movies_test.exs:10
```

Struktur data telah ada, tetapi kita tidak boleh menyimpan data tanpa menggunakan `Repo`.

```elixir
alias Watchlist.Repo
```

```console
1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:9
     Assertion with == failed
     code: response.resp_body() == movie
     lhs:  "[]"
     rhs:  %Watchlist.Movie{__meta__: #Ecto.Schema.Metadata<:built>, id: nil, inserted_at: nil, name: "Back to the future", rating: 5, updated_at: nil}
     stacktrace:
       test/integration/listing_movies_test.exs:17
```

Masih ada beberapa perkara yang perlu dibaiki.  Pertama sekali kita memulangkan satu string, sedangkan kita sepatutnya memulangkan satu senarai wayang.  Baikinya di dalam fail `web/controller/moview_controller.ex`:

```elixir
defmodule Watchlist.MovieController do
  use Watchlist.Web, :controller
  alias Watchlist.Movie

  def index(conn, _params) do
    movies = Repo.all(Movie)
    render conn, movies: movies
  end
end
```

```console
 1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:9
     ** (Plug.Conn.WrapperError) ** (Poison.EncodeError) unable to encode value: {nil, "movies"}
```

Ini bermakna kita cuba untuk mengenkod metadata untuk aplikasi klien, dan `Poison` tidak membenarkannya secara default.  Kita boleh mengatasi keadaaan ini dengan melaksanakan `Poison.Encoder` di dalam model kita.

```elixir
defmodule Watchlist.Movie do
  use Watchlist.Web, :model

  schema "movies" do
    field :name, :string
    field :rating, :integer

    timestamps
  end

  @required_fields ~w(name rating)
  @optional_fields ~w()

  @doc """
  Creates a changeset based on the `model` and `params`.

  If no params are provided, an invalid changeset is returned
  with no validation performed.
  """
  def changeset(model, params \\ :empty) do
    model
    |> cast(params, @required_fields, @optional_fields)
  end

  defimpl Poison.Encoder, for: Watchlist.Movie do
    def encode(movie, _options) do
      movie
      |> Map.from_struct
      |> Map.drop([:__meta__, :__struct__])
      |> Poison.encode!
    end
  end
end
```

Jalankan ujian kita:

```console
1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:9
     Assertion with == failed
     code: response.resp_body() == movie
     lhs:  "[{\"updated_at\":\"2015-12-07T17:08:24Z\",\"rating\":5,\"name\":\"Back to the future\",\"inserted_at\":\"2015-12-07T17:08:24Z\",\"id\":91}]"
     rhs:  %Watchlist.Movie{__meta__: #Ecto.Schema.Metadata<:loaded>, id: 91, inserted_at: #Ecto.DateTime<2015-12-07T17:08:24Z>, name: "Back to the future", rating: 5,
            updated_at: #Ecto.DateTime<2015-12-07T17:08:24Z>}

```

Perhatikan kita mendapat sambutan JSON, tetapi ia gagal.  Apa yang kita harap dipulangkan tidak dienkod.  Enkodkan model yang baru kita bina di dalam ujian integrasi:

```elixir
|> Repo.insert!
|> Poison.encode!
```

Ia akan menghasilkan paparan seperti berikut:

```console
1) test listing movies (ListingMoviesIntegrationTest)
     test/integration/listing_movies_test.exs:9
     Assertion with == failed
     code: response.resp_body() == movie
     lhs:  "[{\"updated_at\":\"2015-12-07T17:11:27Z\",\"rating\":5,\"name\":\"Back to the future\",\"inserted_at\":\"2015-12-07T17:11:27Z\",\"id\":92}]"
     rhs:  "{\"updated_at\":\"2015-12-07T17:11:27Z\",\"rating\":5,\"name\":\"Back to the future\",\"inserted_at\":\"2015-12-07T17:11:27Z\",\"id\":92}"
```

Kita cuba untuk memadankan satu senarai dengan satu string.  Baikinya seperti berikut:

```elixir
defmodule ListingMoviesIntegrationTest do
  use ExUnit.Case, async: true
  use Plug.Test
  alias Watchlist.Router
  alias Watchlist.Movie
  alias Watchlist.Repo

  @opts Router.init([])
  test 'listing movies' do
    %Movie{name: "Back to the future", rating: 5} |> Repo.insert!
    movies = Repo.all(Movie)
             |> Poison.encode!

    conn = conn(:get, "/movies")
    response = Router.call(conn, @opts)

    assert response.status == 200
    assert response.resp_body == movies
  end
end
```

```console
Finished in 0.8 seconds (0.6s on load, 0.1s on tests)
1 test, 0 failures
```

Cuba dengan menggunakan Curl:

```console
$ curl localhost:4000/movies
[{"updated_at":"2015-12-04T21:46:08Z","rating":5,"name":"Ilha das flores","inserted_at":"2015-12-04T21:46:08Z","id":1}]
```

Sumber wayang kita telah siap.  Jalankan ujian keseluruhan untuk memastikan spesifikasi kita berjalan sebagaimana sepatutnya:

```console
$ mix test


  1) test renders 404.json (Watchlist.ErrorViewTest)
     test/views/error_view_test.exs:7
     Assertion with == failed
     code: render(Watchlist.ErrorView, "404.json", []) == %{"errors" => %{"detail" => "Page not found"}}
     lhs:  %{errors: %{detail: "Page not found"}}
     rhs:  %{"errors" => %{"detail" => "Page not found"}}
     stacktrace:
       test/views/error_view_test.exs:8



  2) test render 500.json (Watchlist.ErrorViewTest)
     test/views/error_view_test.exs:12
     Assertion with == failed
     code: render(Watchlist.ErrorView, "500.json", []) == %{"errors" => %{"detail" => "Server internal error"}}
     lhs:  %{errors: %{detail: "Server internal error"}}
     rhs:  %{"errors" => %{"detail" => "Server internal error"}}
     stacktrace:
       test/views/error_view_test.exs:13



  3) test render any other (Watchlist.ErrorViewTest)
     test/views/error_view_test.exs:17
     Assertion with == failed
     code: render(Watchlist.ErrorView, "505.json", []) == %{"errors" => %{"detail" => "Server internal error"}}
     lhs:  %{errors: %{detail: "Server internal error"}}
     rhs:  %{"errors" => %{"detail" => "Server internal error"}}
     stacktrace:
       test/views/error_view_test.exs:18

...

Finished in 0.3 seconds (0.3s on load, 0.06s on tests)
6 tests, 3 failures

```

Ini disebabkan oleh kita menggunakan arahan `--no-html`.  Atasinya dengan membaiki fail `test/views/error_view_test.exs`.

```elixir
defmodule Watchlist.ErrorViewTest do
  use Watchlist.ConnCase, async: true

  # Bring render/3 and render_to_string/3 for testing custom views
  import Phoenix.View

  test "renders 404.json" do
    assert render(Watchlist.ErrorView, "404.json", []) ==
           %{errors: %{detail: "Page not found"}}
  end

  test "render 500.json" do
    assert render(Watchlist.ErrorView, "500.json", []) ==
           %{errors: %{detail: "Server internal error"}}
  end

  test "render any other" do
    assert render(Watchlist.ErrorView, "505.json", []) ==
           %{errors: %{detail: "Server internal error"}}
  end
end

```

Apabila kita mejalankan semula `mix test`, semuanya sepatunya berfungsi dengan sempurna.

## Penutup

Apabila menulis kod, kita perlu yakin dengan apa yang kita buat, dan TDD ialah satu alat yang berkuasa untuk ini.  Maklumbalas yang diberikan memberikan kita penjelasan apabila kita tidak pasti bagaimana untuk membina satu-satu ciri.  Lebih pantas maklumbalas, lebih pantas kita bergerak ke hadapan, walaupun dengan teknologi baru.

Jader Correa berasal dari Brazil, ahli kominiti Elixir dan Ruby, dan mempunyai nyalarasa untuk mengajar, membantu orang lain bermula menulis kod dan juga bermain gitar.
