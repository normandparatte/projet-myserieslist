<template>
  <div>
    <h2>Populaire en ce moment :</h2>
    <md-layout md-gutter>
      <md-layout v-for="(serie , index) in listeSeries" v-bind:key="serie" md-flex-xsmall="100" md-flex-small="50" md-flex-medium="33" md-flex-large="33">
        <md-card class="card">
          <md-card-header>
            <a v-bind:href="'./#/detail/'+serie.id">
              <div class="md-title">{{serie.titre}}</div>
            </a>
          </md-card-header>
          <md-card-media>
            <img class="img-serie-list" v-bind:src="serie.image">
          </md-card-media>
          <md-card-content class="synopsis-list">
            <cite>{{serie.synopsis}}</cite>
          </md-card-content>
        </md-card>
      </md-layout>
    </md-layout>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data () {
    axios.get(`https://amc.ig.he-arc.ch/tmdb/tv/popular?language=fr-CH`)
      .then((response) => {
        let ListeIds = response.data.results.map(item => item.id);
        let ListeTitres = response.data.results.map(item => item.name);
        let ListeImages = response.data.results.map(item => 'http://image.tmdb.org/t/p/w185' + item.poster_path);
        let ListeImagesFond = response.data.results.map(item => 'http://image.tmdb.org/t/p/original' + item.backdrop_path);
        let ListeSynopsis = response.data.results.map(item => item.overview.substr(0, 250) + ' ...');
        let ListeNotes = response.data.results.map(item => item.vote_average / 2);

        let listeSeries = [];
        let lienImage;
        let lienImageFond;

        ListeTitres.forEach((titre, index) => {
          if (typeof (ListeImages[index]) === 'undefined' || ListeImages[index] === null) {
            axios.get(`https://www.googleapis.com/customsearch/v1?cx=011288001747608865807:a7rxzv4srri&q=${titre}&searchType=image&safe=high&key=AIzaSyBlh2KvC84vD0cebFOlMSnLe0-Dx1mc-2A`)
            .then((response) => {
              lienImage = response.data.items.map(item => item.link)[0];
              lienImageFond = lienImage;
              listeSeries.push({
                id: ListeIds[index],
                image: lienImage,
                imageFond: lienImageFond,
                titre: ListeTitres[index],
                synopsis: ListeSynopsis[index],
                note: ListeNotes[index]
              });
              lienImage = null;
              lienImageFond = null;
            });
          } else {
            lienImage = ListeImages[index];
            lienImageFond = ListeImagesFond[index];
            listeSeries.push({
              id: ListeIds[index],
              image: lienImage,
              imageFond: lienImageFond,
              titre: ListeTitres[index],
              synopsis: ListeSynopsis[index],
              note: ListeNotes[index]
            });
            lienImage = null;
            lienImageFond = null;
          }
        });
        this.listeSeries = listeSeries;
      });

    // Remet a zéro le fond
    document.documentElement.style.setProperty('--img', '');

    return {
      listeSeries: []
    };
  },
  watch: {
    listeSeries: {
      handler () {
        localStorage.clear;
        this.listeSeries.forEach(function (serie) {
          localStorage.setItem(serie.id, JSON.stringify(serie));
        }, this);
      },
      deep: true
    }
  },
  props: ['serie'],
  name: 'serie'
};
</script>

<style>
.card {
  max-width: 470px;
  display: inline-block;
  padding: 10px;
  margin: 26px;
}

h2 {
  margin-left: 28px;
  color: #263238;
  font-family: fantasy;
  font-size: 1.9em !important;
}

.md-title {
  text-align: center;
  font-size: 2.4em !important;
  font-family: fantasy;
}

a:hover {
  text-decoration: none !important;
  color: #80CBC4 !important;
}

.img-serie-list {
  width: inherit !important;
  margin: auto !important;
  display: block !important;
}

.synopsis-list {
  color: whitesmoke;
  font-size: 1.2em !important;
}
</style>
