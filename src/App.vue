<template>
  <div id="app">
    <div style="text-align: left">
      <h3>Nearest airports to {{ selectedAirport}} :</h3>
      <table border="1">
        <thead>
        <tr>
          <th>Airport</th>
          <th>Distance (km)</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="airport in nearest">
          <td>{{ airport.lid }}</td>
          <td>{{ airport.distance }} </td>
        </tr>
        </tbody>
      </table>
    </div>
    <div>
      <vue-good-table
        :columns="columns"
        :rows="rows"
        styleClass="table table-bordered table-striped condensed"
        :paginate="true"
        :globalSearch="true"
        :lineNumbers="false">
        <template slot="table-row" slot-scope="props">
          <td>{{ props.row.City }}</td>
          <td>{{ props.row.LocationID }}</td>
          <td>{{ props.row.Lat }}</td>
          <td>{{ props.row.Lon }}</td>
          <td>
            <button
              v-on:click="nearest = findNearest(props.row.LocationID, props.row.Lat, props.row.Lon); selectedAirport=props.row.City ">
              Find Nearest
            </button>
          </td>
        </template>
      </vue-good-table>
    </div>
  </div>
</template>
<script>
  import alaska from '../src/assets/alaska_airports.json'
  import Vue from 'vue'
  import VueGoodTable from 'vue-good-table'
  import VueLodash from 'vue-lodash'
  Vue.use(VueLodash)
  Vue.use(VueGoodTable)
  export default {
    name: 'app',
    methods: {
      haversineFormula: function (lat1, lon1, lat2, lon2) {
        if (Number.prototype.toRadians === undefined) {
          Number.prototype.toRadians = function () {
            return this * Math.PI / 180;
          }
        }
        var R = 6371000; // metres
        var φ1 = lat1.toRadians();
        var φ2 = lat2.toRadians();
        var Δφ = (lat2 - lat1).toRadians();
        var Δλ = (lon2 - lon1).toRadians();
        var a = Math.sin(Δφ / 2) * Math.sin(Δφ / 2) + Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
        var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
        var d = R * c;
        //toRadians is not implemented
        // radians = degrees * Math.PI / 180
        return d
      },
      findNearest: function (src_loc_id, src_lat, src_lon) {
        var vm = this
        var distances = alaska.map(function (value, key) {
          return {
            lid: value.LocationID,
            distance: _.round(vm.haversineFormula(src_lat, src_lon, value.Lat, value.Lon) / 1000)
          }
        })
        return _.sortBy(distances, [function (o) {
          return o.distance;
        }]).slice(1, 4)
      }
    },
    data () {
      return {
        nearest: [],
        selectedAirport: 'none',
        columns: [
          {
            label: 'City',
            field: 'City'
          },
          {
            label: 'LocationID',
          },
          {
            label: 'Lat',
          },
          {
            label: 'Lon',
          },
          {
            label: ""
          }
        ],
        rows: alaska,
      }
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
</style>
