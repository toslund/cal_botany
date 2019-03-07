<template>
  <v-container>
    <v-layout
      text-xs-center
      wrap
    >
      <v-flex xs12>
        <v-img
          :src="require('../assets/poppy.png')"
          class="my-3"
          contain
          height="200"
        ></v-img>
      </v-flex>

      <v-flex mb-4>
        <h1 class="display-2 font-weight-bold mb-3">
          Learn California Plants
        </h1>
      </v-flex>

      <v-flex
        mb-5
        xs12
      >
        <h2 class="headline font-weight-bold mb-3">What is this?</h2>

        <v-layout justify-center>
          <p>A free tool to help anyone learn California plants with interactive quizes.</p>
        </v-layout>
      </v-flex>

      <v-flex
        xs12
        mb-5
      >
        <h2 class="headline font-weight-bold mb-3">Who is it for?</h2>

        <v-layout justify-center>
          <p>Ecologists, botanists, students, outdoor enthusiasts,
            and anyone else who is interested in California's biodiversity.</p>
        </v-layout>
      </v-flex>

      <v-flex
        xs12
        mb-5
      >
        <h2 class="headline font-weight-bold mb-3">A Big Thanks to</h2>

        <v-layout justify-center>
          The community at<a href="https://www.inaturalist.com" class="subheading mx-3">iNaturalist</a>for
          their dedication and observations that are the foundation of this app.
        </v-layout>
      </v-flex>
    </v-layout>
  </v-container>
</template>

