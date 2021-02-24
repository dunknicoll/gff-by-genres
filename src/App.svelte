<script>
  import { onMount } from "svelte";
  import * as hpq from "hpq";
  let movies = [];
  let genres = [];
  let selectedGenre = "";
  $: moviesOrganised = movies
    .filter(item => {
      if (selectedGenre === "") return true;
      return item.genres.includes(selectedGenre);
    })
    .sort((movieA, movieB) => {
      if (movieA.title < movieB.title) return -1;
      return 1;
    });

  const loadData = async () => {
    const raw = await fetch(
      "https://athome.glasgowfilm.org/page/all-gff-films/"
    );
    const text = await raw.text();
    const rawData = hpq.parse(text, {
      genres: hpq.query(".meta-item-tagline-item:first-child", hpq.text()),
      titles: hpq.query(".caption .title", hpq.html()),
      alts: hpq.query(".caption .title", hpq.text()),
      images: hpq.query(".poster .poster-image", hpq.attr("src")),
      links: hpq.query(".collection-items .meta-item-link", hpq.attr("href"))
    });

    genres = Array.from(
      new Set(
        rawData.genres
          .map(genre =>
            genre
              .split(", ")
              .map(word => `${word[0].toUpperCase()}${word.substr(1)}`)
          )
          .flat()
      )
    ).sort();

    movies = rawData.titles.map((title, index) => ({
      title,
      alt: rawData.alts[index],
      genres: rawData.genres[index]
        .split(", ")
        .map(word => `${word[0].toUpperCase()}${word.substr(1)}`),
      image: rawData.images[index],
      link: `https://athome.glasgowfilm.org${rawData.links[index]}`
    }));
  };

  onMount(async () => {
    await loadData();
  });

  const setSelectedGenre = (genre = "") => {
    if (genre === selectedGenre) {
      selectedGenre = "";
    } else {
      selectedGenre = genre;
    }
  };
</script>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Raleway:wght@400;600;700&display=swap");
  :global(body) {
    background-color: black;
    color: white;
    font-family: "Raleway", sans-serif;
    box-sizing: border-box;
  }
  h1 {
    margin-left: auto;
    margin-right: auto;
    width: 95%;
  }
  nav {
    margin: 0 auto;
    width: 94%;
    margin-bottom: 20px;
    display: grid;
    grid-template-columns: repeat(8, 1fr);
  }
  .nav-item {
    cursor: pointer;
    margin-bottom: 5px;
    margin-right: 20px;
  }
  .nav-link.active {
    font-weight: bold;
    border-bottom: 1px solid white;
  }
  .nav-link:hover {
    border-bottom: 1px dashed lightgrey;
  }
  .selected-genre {
    margin: 0 auto;
    width: 94%;
    font-weight: bold;
    font-size: 25px;
    margin-bottom: 5px;
  }
  main {
    margin: 0 auto;
    width: 95%;
    display: grid;
    grid-template-columns: 25% 25% 25% 25%;
  }
  .movie {
    margin: 0 auto;
    width: 95%;
    margin-bottom: 20px;
  }
  .title {
    font-size: 16px;
    font-weight: bold;
  }
  .title a {
    color: white;
  }
  .poster-image {
    width: 100%;
    border-radius: 5px;
  }
  .clear-genre {
    color: lightgrey;
    display: inline-block;
    border-left: 1px solid white;
    padding-left: 10px;
    cursor: pointer;
  }
  .clear-genre:hover {
    color: white;
  }
  .hide {
    display: none;
  }
</style>

<h1>GFF by Genres</h1>

<nav>
  {#each genres as genre}
    <div class="nav-item">
      <span
        class="nav-link"
        class:active={selectedGenre === genre}
        on:click={() => setSelectedGenre(genre)}>
        {genre}
      </span>
    </div>
  {/each}
</nav>

<div class="selected-genre">
  {selectedGenre || 'All Genres'}
  <span
    on:click={() => setSelectedGenre()}
    class="clear-genre"
    class:hide={selectedGenre === ''}>
    x
  </span>
</div>

<main>
  {#each moviesOrganised as movie}
    <div class="movie">
      <div class="poster">
        <a href={movie.link} target="_blank">
          <img class="poster-image" src={movie.image} alt={movie.alt} />
        </a>
      </div>
      <div class="title">
        <a href={movie.link} target="_blank">
          {@html movie.title}
        </a>
      </div>
    </div>
  {/each}
</main>
