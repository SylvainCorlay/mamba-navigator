<template>
  <v-app>

    <v-content>
      <HelloWorld/>

    <v-row>
    <v-col
      cols="12"
      sm="4"
    >
    <v-card>
       <v-list>
         <v-subheader>ENVIRONMENTS</v-subheader>
         <v-list-item-group
           v-model="selectedEnvIndex"
           color="primary">
           <v-list-item
             v-for="env in envs"
             :key="env"
           >
             {{ env }}
           </v-list-item>
         </v-list-item-group>
       </v-list>
    </v-card>
    </v-col>

    <v-col
      cols="12"
      sm="8"
    >
    <v-card>
      <p>Selected environment (name): {{ selectedEnvName }}</p>
    </v-card>
    <v-card>
      <h3>Select a package and view its dependencies in a graph!</h3>
    </v-card>
    <v-card>
      <v-data-table
        v-model="selectedPkg"
        :headers="columns"
        :items="items"
        :single-select=false
        item-key="name"
        show-select
        class="elevation-1"
      >
        <template v-slot:top>
        </template>
      </v-data-table>
    </v-card>
    <v-card>
      <Network :data-promise="depData" :key="componentKey"></Network>
    </v-card>
    </v-col>
    </v-row>
    </v-content>
  </v-app>
</template>

<script>
import axios from 'axios';
import HelloWorld from './components/HelloWorld';
import Network from './components/Network';

export default {
  name: 'App',

  components: {
    HelloWorld,
    Network
  },

  data: () => ({
    envs: [],
    items: [],
    selectedEnvIndex: [],
    selectedEnvName: '',
    selectedPkg: [],
    selectedPkgName: [],
    envUrl: 'http://0.0.0.0:5000/envs',
    pkgUrl: 'http://0.0.0.0:5000/pkgs',
    depUrl: [],
    depData: [],
    componentKey: 0,
    columns: [{text: "Package", value: "name"},
              {text: "Version", value: "version", sortable: false},
              {text: "Platform", value: "platform"}]
  }),
  watch: {
    // whenever selectedEnvIndex changes, this function will run
    selectedEnvIndex: function () {
      this.getEnvName()
    },
    // whenever selectedPkg changes, this function will run
    selectedPkg: function () {
      this.getPkgName()
    },
    // whenever selectedEnvName changes, this function will run
    selectedEnvName: function () {
      this.getPkgs()
    }
  },

  mounted: function() {
    this.getEnvs();
  },
  methods: {
    getEnvs() {
      let url = this.envUrl;
      axios.get(url).then((response) => {
        this.envs = response.data.envs;
      }).catch(error => { console.log(error); });
    },
    getEnvName() {
      let name = this.envs[this.selectedEnvIndex].split('/').pop();
      if (name.includes('miniconda') || name.includes('anaconda')) {
        this.selectedEnvName = 'base';
      } else {
        this.selectedEnvName = name;
      }
    },
    getPkgs() {
      let reqUrl = this.envUrl + '/' + this.selectedEnvName;
      axios.get(reqUrl).then((response) => {
        this.items = response.data;
      }).catch(error => { console.log(error); });
    },
    getPkgName() {
      this.componentKey += 1;
      let pkg = this.selectedPkg[0];
      if (pkg && pkg.name) {
        let pkgNames = this.selectedPkg.map(function (i) { return i.name; });
        this.selectedPkgName = pkgNames;
      } else {
        this.selectedPkgName = ['python'];
      }
      let url = this.pkgUrl;
      this.depUrl = this.selectedPkgName.map(function (i) { return url + '/' + i });
      this.depData = this.depUrl.map(u => fetch(u).then(resp => resp.json()));
    },
  },
};
</script>
