<template>
  <v-container>
      <v-alert type="info" prominent>Denne siden er ikke ferdig. Ting som skal legges inn;
          <ul>
            <li>Antall seire</li>
            <li>Vinn/tap-prosent</li>
            <li>Annet...?</li>
          </ul>
      </v-alert>
    <h1>Statistikk</h1>
    <v-btn to="/">Tilbake</v-btn>
    <v-data-table
        :headers="table_spillere_headers"
        :items="spillere"
        disable-pagination
        disable-filtering
        :mobile-breakpoint="0"
        hide-default-footer
        :items-per-page="-1">
    </v-data-table>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      data: [],
      spillere: [],
      data_spiller: {},
      events: [],
      cal_model: '',
      selectedEvent: '',
      table_spillere_headers: [
        {text: "Navn", align: "left", sortable: true, value: "navn"},
        {text: "Antall spill", align: "left", sortable: true, value: "antall_spill"},
      ]
    };
  },
  mounted() {
    let vm = this;
    axios.get("/spill.json").then(res => {
      let data = res.data.filter(n => n != null && "dato" in n).reverse();
      data.forEach(spill => {
        spill.poengtavle.forEach(sett => {
          if (vm.data_spiller[sett.navn] == undefined)
            vm.data_spiller[sett.navn] = [];
          vm.data_spiller[sett.navn].push({
            dato: spill.dato,
            navn: spill.navn,
            poeng: sett.poeng
          });
        });
      });

      vm.spillere = Object.keys(vm.data_spiller).map(navn => {
          return {
              navn,
              antall_spill: vm.data_spiller[navn].length,
              spill: vm.data_spiller[navn]
          }
      });

      vm.data = data;

      vm.events = data.map(spill => {
        let start = (new Date(spill.dato));
        let slutt = (new Date(spill.dato)).setHours(start.getHours() + 1);
        return {
          name: spill.navn,
          start,
          slutt,
          poengtavle: spill.poengtavle
        }
      })

    });
  },
  methods: {
    selectEvent({ event }) {
      this.selectedEvent = event;
    }
  }
};
</script>

<style scoped>
.v-card {
  margin-bottom: 20px;
}
</style>
