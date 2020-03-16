<template>
  <v-layout
    justify-center
    align-center
    fluid
  >
    <v-flex
      xs12
      sm8
      md6
      lg4
    >
      <div class="flex title font-weight-black">NHL - Upcoming Events</div>
      <v-list 
        class="transparent"
        v-for="line in lines"
        :key="line.home_team" 
      >
        <v-list-item>
          <span class="overline white--text indigo text--{lighten}-{2}">{{ line.start }}</span>
          <span class="pl-md-2">
            <a>
              Bet
            </a>
          </span>
        </v-list-item>

        <v-list-item>
          <v-list-item-title>{{ line.teams[0] }}</v-list-item-title>
          <v-list-item-subtitle class="text-right">{{ line.odds[0] }}</v-list-item-subtitle>
        </v-list-item>

        <v-list-item>
          <v-list-item-title>{{ line.teams[1] }}</v-list-item-title>
          <v-list-item-subtitle class="text-right">{{ line.odds[1] }}</v-list-item-subtitle>
        </v-list-item>

        <v-divider></v-divider>
      </v-list>



      <v-row justify="center">
        <v-dialog v-model="dialog" persistent max-width="290">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark v-on="on">Open Dialog</v-btn>
          </template>
          <v-card>
            <v-card-title class="headline">Use Google's location service?</v-card-title>
            <v-card-text>Let Google help apps determine location. This means sending anonymous location data to Google, even when no apps are running.</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="green darken-1" text @click="dialog = false">Disagree</v-btn>
              <v-btn color="green darken-1" text @click="dialog = false">Agree</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-row>
    </v-flex>
  </v-layout>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      api: {
        key: process.env.API_KEY,
        bases: {
          sports: 'https://api.the-odds-api.com/v3/sports/',
          odds: 'https://api.the-odds-api.com/v3/odds/'
        }
      },
      lines: [],
      dialog: false
    }
  },
  mounted () {
    //this.fetchLines('mma_mixed_martial_arts', 'us');
    this.getSports()
  },
  methods: {
    loadData() {
      this.getSports();
    },
    getSports: function () {
      try {
        let uri = `${this.api.bases.sports}?apiKey=${this.api.key}`;
        axios
          .get(`/sports.json`)
          .then(r => {
            console.log(r)
          })
      } catch (err) {
        console.log(err);
      }
    },
    fetchLines: function (s, r) {
      //let uri = `${this.base}?apiKey=${this.key}&sport=${s}&region=${r}`;
      let uri = '/test.json';
      axios
        .get(uri)
        .then(r => this.formatLines(r.data.data))
    },
    formatLines: function (lines) {
      let arr = [];
      for (const line of lines) {
        try {
          this.lines.push({
            start: this.formatTime(line.commence_time),
            odds: this.formatOdds(line.sites),
            teams: line.teams
          });
        } catch (err) {
          console.log(err);
        }
      }
    },
    formatTime: function(timestamp) {
      try {
        // multiplied by 1000 so that the argument is in milliseconds, not seconds.
        return new Date(timestamp * 1000).toLocaleString();
      } catch (err) {
        console.log(err);
        return 'n/a';
      }
    },
    formatOdds: function (sites) {
      if (sites && sites[0] && sites[0].odds) {
        let odds = sites[0].odds.h2h;
        return this.convertToAmericanOdds(odds[0], odds[1]);
      } else {
        return [
          'n/a',
          'n/a'
        ];
      }
    },
    convertToAmericanOdds: function (a, b) {
      if (a == Math.max(a, b)) {
        a = this.formatHigh(a);
        b = this.formatLow(b);
      } else {
        a = this.formatLow(a);
        b = this.formatHigh(b);
      }

      return [a, b];
    },
    formatHigh(n) {
      return parseFloat((n - 1) * 100).toFixed(2);
    },
    formatLow(n) {
      return parseFloat(100 / (1 - n)).toFixed(2);
    }
  }
}
</script>


<style scoped>



</style>