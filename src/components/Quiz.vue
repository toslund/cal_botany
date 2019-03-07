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
            Customize your Study Session
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
      wrap
      v-if="step == 'guess'">
      <v-flex xs12>
    <v-card
            v-for="(val, idx) in visiblePlant"
            :key="idx"
            v-show="idx == 1"
          >
            <v-card-text>
              <p> {{idx}} </p>
          <v-carousel
            :cycle="false"
            v-model="carouselIndex"
            >
            <v-carousel-item
              v-for="(value, i) in val.ninePhotos"
              :key="i"
              :src="value.medium_url"
            ></v-carousel-item>
          </v-carousel>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-icon color="primary" dark v-on="on">copyright</v-icon>
            </template>
            <!--<span>iNaturalist: {{ eightPhotos[carouselIndex].attribution }}</span>-->
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
          <div>
            <v-btn
              @click="submit()"
              color="success"
              :disabled="!val.guess || val.submitted"
            >Submit
            </v-btn>
            <v-btn
              @click="next()"
              color="success"
              :disabled="!val.submitted">Next</v-btn>
            <v-dialog
              v-model="dialog"
              max-width="400"
              v-if="dialog"
            >
              <v-card>
                <v-img
                  aspect-ratio="1"
                  :src="dialogPhoto.medium_url"
                ></v-img>
                <v-card-title class="justify-center headline font-weight-bold">{{ dialogSpecies }}</v-card-title>
                <v-card-text>
                  <div class="text-xs-center"><img style="max-width:100px;" :src="guessCorrect ? require('../assets/good.png') : require('../assets/bad.png')"></div>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="green darken-1"
                    @click="nextRound()"
                  >
                    Next
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </div>
          <div>
            Score: {{ scoreRight }}/{{ scoreTotal }}
          </div>
        </v-card-text>
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
        'All Species in California!': null
      },
      list: null,
      quizData: [],
      carouselIndex: 0,
      plantIndex: 0
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
      this.carouselIndex = 0;
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
      } else { return [this.quizData[this.plantIndex - 1], this.quizData[this.plantIndex], this.quizData[this.plantIndex + 1]]; }
    }
  }
}
</script>

<style>

</style>
