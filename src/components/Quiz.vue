<template>
  <v-container>
    <v-layout
      text-xs-center
      wrap
      v-if="step == 'customize'"
    >
      <v-flex xs12>
        <v-card>
          <v-card-title class="headline font-weight-regular light-green lighten-4">
            Choose a Plant List
          </v-card-title>
            <v-card-text>
            <v-autocomplete
              v-model="list"
              clearable
              :hint="'Choose a plant list to study'"
              persistent-hint
              :items="listChoices"
              prepend-icon="local_florist"
            >
            </v-autocomplete>
            <v-btn @click="makeQuiz()"
              color="#689F38"
              :disabled="!list"
              class="white--text"
            >Begin</v-btn>
            </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
    <v-layout
      text-xs-center
      justify-center
      wrap
      v-if="step == 'guess' && !finished">
      <v-flex xs12 md8 lg6>
        <v-card
          v-for="(val, idx) in visiblePlant"
          :key="idx"
          v-show="idx == 1">
        <v-card-text>
            <v-carousel
              :cycle="false"
              v-model="carouselIndex"
              >
              <v-carousel-item
                v-for="(value, i) in val.ninePhotos"
                :key="i"
              >
                <div style="background-color: #D3D3D3; height: 100%">
                  <img
                    style="height:100%"
                    :src="value.medium_url"
                  >
                </div>
              </v-carousel-item>
            </v-carousel>
            <v-tooltip bottom v-if="val.ninePhotos">
              <template v-slot:activator="{ on }">
                <v-icon color="primary" dark v-on="on">copyright</v-icon>
              </template>
              <span>iNaturalist: {{ val.ninePhotos[carouselIndex].attribution }}</span>
            </v-tooltip>

            <div v-show="!val.submitted">
              <v-radio-group v-model="val.guess">
              <v-radio
                v-for="n in val.choices"
                :key="n"
                :label="n"
                :value="n"
              ></v-radio>
            </v-radio-group>
            </div>
            <div v-show="val.submitted">
              <p class="justify-center headline font-weight-bold">{{ val.sn }}</p>
              <div class="text-xs-center">
                <img
                  style="max-width:100px;"
                  :src="val.result ? require('../assets/good.png') : require('../assets/bad.png')">
              </div>
            </div>
            <div
            v-if="!finished">
              <v-btn
                @click="previous()"
                color="teal lighten-4"
                :disabled="plantIndex == 0"
                >Previous</v-btn>
              <v-btn
                @click="submit()"
                color="green darken-2"
                :disabled="!val.guess || val.submitted"
              >Submit
              </v-btn>
              <v-btn
                @click="next()"
                color="teal lighten-4"
                v-if="plantIndex < totalPlants - 1"
                :disabled="!val.submitted">Next</v-btn>
            </div>
            <div
            v-else>
              <v-btn
                @click="seeResults()"
                color="success"
                :disabled="!val.submitted">Review Results</v-btn>
            </div>
            <div>
              {{ plantIndex }} of {{ totalPlants }}
            </div>
          </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
      <v-layout
      text-xs-center
      justify-center
      wrap
      v-else>
        <v-flex xs12 md8 lg6>
          <v-card>
            <v-expansion-panel>
              <v-expansion-panel-content
                v-for="(item,i) in quizData"
                :key="i"
              >
                <template v-slot:header>
                  <div><v-icon :color="item.result ? 'green' : 'red'" dark v-on="on">
                    {{ item.result ? 'thumb_up' : 'thumb_down' }}</v-icon>
                    {{ item.sn }}
                  </div>
                </template>
                <v-card>
                  <v-container grid-list-sm fluid>
                    <v-layout row wrap>
                      <v-flex
                        v-for="(photo, i) in item.ninePhotos"
                        :key="i"
                        xs4
                        d-flex
                      >
                    <v-card flat tile class="d-flex">
                    <v-img
                      :src="photo.medium_url"
                      aspect-ratio="1"
                      class="grey lighten-2"
                    >
                      <template v-slot:placeholder>
                        <v-layout
                          fill-height
                          align-center
                          justify-center
                          ma-0
                        >
                          <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
                        </v-layout>
                      </template>
                    </v-img>
                    </v-card>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-card>
        </v-flex>
      </v-layout>
  </v-container>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Quiz',
  data: function() {
    return {
      step: 'customize',
      studyLists: {
        'Falcons of Southern California': ['Falco sparverius', 'Haliaeetus leucocephalus',
          'Aquila chrysaetos', 'Falco peregrinus', 'Circus hudsonius', 'Elanus leucurus',
          'Parabuteo unicinctus', 'Accipiter cooperii', 'Buteo lineatus', 'Buteo jamaicensis',
          'Accipiter striatus', 'Buteo swainsoni', 'Tyto alba', 'Athene cunicularia',
          'Bubo virginianus', 'Megascops kennicottii', 'Pandion haliaetus', 'Cathartes aura'],
        'Coastal Sage Scrub (Venturan/Diegan)': ['Artemisia californica', 'Eriogonum fasciculatum',
          'Diplacus aurantiacus', 'Isocoma menziesii', 'Opuntia littoralis'],
        'Oaks of Mendocino County': ['Quercus douglasii', 'Quercus garryana', 'Quercus lobata', 'Quercus agrifolia',
          'Quercus kelloggii', 'Quercus parvula', 'Quercus parvula shrevei', 'Quercus wislizeni', 'Quercus chrysolepis'],
        'Orange County Invasive Priorities (NCC Priority I&II)': ['Acacia cyclops', 'Acacia redolens', 'Aegilops triuncialis',
          'Ageratina adenophora', 'Ailanthus altissima', 'Albizia lophantha',
          'Araujia sericifera', 'Arctotheca calendula', 'Arundo donax', 'Asparagus asparagoides',
          'Asphodelus fistulosus', 'Brassica tournefortii', 'Cenchrus echinatus', 'Cenchrus longispinus',
          'Centaurea diluta', 'Centaurea solstitialis', 'Chrysanthemoides monilifera', 'Cirsium vulgare',
          'Conium maculatum', 'Cortaderia jubata', 'Cortaderia selloana', 'Cynara cardunculus', 'Delairea odorata',
          'Dittrichia graveolens', 'Echium candicans', 'Ehrharta calycina', 'Ehrharta longiflora', 'Emex spinosa',
          'Erodium malacoides', 'Euphorbia terracina', 'Euphorbia virgata', 'Ficus carica', 'Foeniculum vulgare',
          'Galenia pubescens', 'Gazania linearis', 'Glebionis coronaria', 'Hypericum canariense',
          'Iris pseudacorus', 'Kochia scoparia', 'Lepidium appelianum', 'Lepidium draba', 'Lepidium latifolium',
          'Leucanthemum vulgare', 'Ligustrum japonicum', 'Limonium duriusculum', 'Limonium ramosissimum',
          'Lonicera japonica', 'Malephora crocea', 'Melia azedarach', 'Melinis repens', 'Nassella tenuissima',
          'Olea europaea', 'Oncosiphon piluliferum', 'Parkinsonia aculeata', 'Parthenium hysterophorus',
          'Parthenocissus quinquefolia', 'Pennisetum setaceum', 'Phalaris aquatica', 'Plantago arenaria',
          'Plecostachys serpyllifolia', 'Ricinus communis', 'Robinia pseudoacacia', 'Rubus armeniacus',
          'Salpichroa origanifolia', 'Schinus molle', 'Schinus terebinthifolius', 'Senecio linearifolius',
          'Spartium junceum', 'Tamarix aphylla Athel', 'Tamarix ramosissima', 'Tropaeolum majus',
          'Ulmus parvifolia', 'Verbesina encelioides', 'Vinca major', 'Volutaria tubuliflora',
          'Washingtonia filifera', 'Washingtonia robusta'],
        'Lilies of Orange County': ['Calochortus albus', 'Calochortus splendens', 'Calochortus catalinae',
          'Calochortus invenustus', 'Calochortus plummerae', 'Calochortus weedii', 'Lilium humboldtii'],
        // 'All Species in California!': null
      },
      list: null,
      quizData: [],
      carouselIndex: 0,
      plantIndex: 0,
    }
  },
  methods: {
    fetch: function (speciesIdx) {
      var speciesName = this.plants[speciesIdx]
      console.log('fetching');
      var obsURL = "https://www.inaturalist.org/observations.json?";
      var params = new URLSearchParams({
        "taxon_name": speciesName,
        "has[]": "photos",
        "quality_grade": "research"
      });
      var searchURL = obsURL + params.toString();
      console.log(searchURL);
      axios
        .get(searchURL)
        .then(response => (this.clean(speciesIdx, response.data)))
    },
    clean: function (speciesIdx, data) {
      // var compiledPhotos = [];
      this.info = data;
      console.log('cleaning');
      for (var i = 0; i < data.length; i++) {
        if (data[i].taxon.name === this.quizData[speciesIdx].sn) {
          for (var photo in data[i].photos) {
            if (data[i].photos[photo].license_code.slice(0, 2) === 'CC') {
              this.quizData[speciesIdx].photos.push(data[i].photos[photo]);
            }
          }
        }
        if (this.quizData[speciesIdx].photos.length >= 9 ) {
          this.quizData[speciesIdx].ninePhotos = this.jumbleArray(this.quizData[speciesIdx].photos).slice(-9);
        } else { this.quizData[speciesIdx].ninePhotos = this.jumbleArray(this.quizData[speciesIdx].photos); }
        // this.$set(this.fetched, species, compiledPhotos);
      }
    },
    makeQuiz: function () {
      for (var i in this.plants) {
        console.log(this.plants[i]);
        console.log(i);
        this.quizData[i].choices = this.getChoices(this.plants[i])
        this.fetch(i);
        this.step = 'guess';
      }
    },
    jumbleArray: function (inArray) {
      var a = inArray.slice();
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [a[i], a[j]] = [a[j], a[i]];
      }
      return a;
    },
    getChoices: function (correctPlant) {
      let a = this.plants.slice();
      let idx = a.indexOf(correctPlant);
      a.splice(idx, 1);
      let jumbledA = this.jumbleArray(a);
      let choices = jumbledA.slice(-3);
      choices.push(correctPlant);
      return this.jumbleArray(choices);
    },
    submit: function () {
      if (this.quizData[this.plantIndex].guess === this.quizData[this.plantIndex].sn) {
        this.quizData[this.plantIndex].result = true;
      } else {this.quizData[this.plantIndex].result = false;}
      this.quizData[this.plantIndex].submitted = true
    },
    next: function() {
      this.plantIndex += 1;
      // for (var i; i < 100; i += 1) {
      //   if (this.carouselIndex == 0) { break }
      //   else if (this.carouselIndex == this.visiblePlant[1].photos.length - 1) {
      //     this.carouselIndex += 1;
      //     break
      //   }
      //   else {this.carouselIndex += 1;}
      // }
      // this.carouselIndex = 0;
    },
    previous: function() {
      this.plantIndex -= 1;
      this.carouselIndex = 0;
    },
    seeResults: function() {
      //
    }
  },
  computed: {
    listChoices: function() {
      return Object.keys(this.studyLists)
    },
    plants: function () {
      if (!this.list) { return null }
      var species = this.studyLists[this.list];
      if (!species) {
        return null; // Then get random from all species
      }
      var jumbledSpecies = this.jumbleArray(species);
      for (var i in jumbledSpecies) {
        this.quizData.push({
          sn: jumbledSpecies[i],
          photos: [],
          ninePhotos: [],
          choices: [],
          guess: null,
          submitted: false,
          result: null
        })
        // this.$set(jumbledSpecies[i], this.quizData[this.chosenType][taxon], {
        //   "photos": [],
        //   "guess": null
        // });
      }
      return jumbledSpecies
    },
    visiblePlant: function() {
      if (this.plantIndex == 0) {
        return [[], this.quizData[this.plantIndex], this.quizData[this.plantIndex + 1]];
      } else if (this.plantIndex == this.totalPlants - 1) {
        return [this.quizData[this.plantIndex - 1], this.quizData[this.plantIndex]];
      } else { return [this.quizData[this.plantIndex - 1], this.quizData[this.plantIndex], this.quizData[this.plantIndex + 1]]; }
    },
    totalPlants: function() {
      var total = 0;
      if (!this.plants) { return total; }
      return this.plants.length;
    },
    totalCorrect: function() {
      var totalCorrect = 0;
      if (!this.quizData) { return total; }
      for (var plant in this.quizData) {
        if (this.quizData[plant].result == true) { totalCorrect += 1; }
      }
      return totalCorrect;
    },
    totalGuessed: function() {
      var totalCorrect = 0;
      if (!this.quizData) { return total; }
      for (var plant in this.quizData) {
        if (this.quizData[plant].submitted == true) { totalCorrect += 1; }
      }
      return totalCorrect;
    },
    finished: function() {
      if ( this.totalGuessed >= this.totalPlants ) { return true }
      else { return false }
    }
  }
}
</script>

<style>

</style>
