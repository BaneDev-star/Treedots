<template>
  <q-page class="flex flex-center page-container">
    <q-google-map :center="{
    lat: center.lat,
    lng: center.lng
    }" :zoom="13" :style="{width: '100%', height: '100%'}" :options="mapOptions">
      <q-google-map-marker :icon="mylocationSVG" :position="center" :draggable="false"></q-google-map-marker>
      <q-google-map-marker v-for="m in hubs" v-bind:key="m.id" @click="clickMarker(m)" :position="m.position"
                           :clickable="true"
                           :draggable="false" :icon="hubIcon(m)" :label="hubLabel(m)">
      </q-google-map-marker>
    </q-google-map>
    <q-list class="hub-list">
      <q-item-label
        header
        class="text-grey-8"
      >
        Hubs Near You
      </q-item-label>

      <q-item
        clickable
        tag="a"
        target="_blank"
        v-for="(m) in filteredHubs"
        v-bind:key="m.id"
      >
        <div class="hub-item">
          <q-item-label>{{ m.name }} {{ m.distance }}</q-item-label>
          <svg width="40" height="40" xmlns="http://www.w3.org/2000/svg">
            <g id="Layer_1">
              <title>Layer 1</title>
              <ellipse stroke-width="4" ry="15" rx="15" id="svg_1" cy="20" cx="20" stroke="#ffffff" fill="#c18c30" />
              <text transform="matrix(1 0 0 1 0 0)" xml:space="preserve" text-anchor="start" font-family="Noto Sans JP"
                    font-size="18" id="svg_2" y="26.07468" x="13.50075" stroke-width="0" stroke="#ffffff"
                    fill="#ffffff">{{m.marker_title}}</text>
            </g>
          </svg>
        </div>
      </q-item>
    </q-list>
  </q-page>

</template>

<script>
  import hubsJson from '../assets/hubs.json';
  import axios from "axios";
  import mylocationSVG from '../assets/mylocation.svg';
  import hubMarker from '../assets/hubmarker.svg';

  export default {
    name: 'PageIndex',
    data() {
      return {
        inner_height: 0,
        center: {
          lat: 1.330376,
          lng: 103.820413
        },
        mapOptions: {
          mapTypeControl: false,
          streetViewControl: false,
          zoomControl: false,
          fullscreen: false,
        },
        hubs: hubsJson['data'],
        mylocationSVG: mylocationSVG,
        selectedHub: null
      }
    },
    created() {
      this.inner_height = window.innerHeight - 95;
    },

    async mounted() {
      this.getCurrentPosition();
    },

    computed: {
      filteredHubs() {
        if (this.selectedHub) {
          return this.hubs.filter(item => item.id !== this.selectedHub.id);
        }
        return this.hubs;
      },
    },

    methods: {
      clickMarker(obj) {
        if (this.selectedHub && this.selectedHub.id === obj.id) {
          this.selectedHub = null;
        } else {
          this.selectedHub = obj;
        }
      },
      mouseoverMarker(obj) {
        this.infoWinOpen = true;
      },
      mouseoutMarker(obj) {
        this.infoWinOpen = false;
      },
      getCurrentPosition() {
        navigator.geolocation.getCurrentPosition(
          position => {
            this.center = {
              lat: position.coords.latitude,
              lng: position.coords.longitude
            };
            this.updateHubData();
          },
          error => {
            console.log(error.message);
          },
        );
      },

      async updateHubData() {
        for (let i = 0; i < this.hubs.length; i++) {
          const item = this.hubs[i];
          item.position = await this.getHubLatLng(item.name);
          item.distance = 0;
          if (item.position) {
            item.distance = Math.pow(item.position.lat - this.center.lat, 2) + Math.pow(item.position.lng - this.center.lng, 2);
          }
          this.hubs[i] = item;
        }

        this.hubs.sort((a, b) => {
          return a.distance < b.distance ? -1 : a.distance > b.distance ? 1 : 0;
        });

        for (let i = 0; i < this.hubs.length; i++) {
          this.hubs[i].marker_title = this.hubTitle(i);
        }
      },

      async getHubLatLng(address) {
        try {
          var { data } = await axios.get(
            `https://maps.googleapis.com/maps/api/geocode/json?address=${address}&key=AIzaSyAW_DFG11Jzk_xFtl0MdX_vN0ykzNdUOA0`
          );
          return data.results[0].geometry.location;
        } catch (error) {
          console.log('error = ', error);
          return null;
        }
      },

      hubTitle(index) {
        return String.fromCharCode(65 + index);
      },

      hubIcon(m) {
        if (this.selectedHub && this.selectedHub.id === m.id) {
          return null;
        }
        return hubMarker;
      },

      hubLabel(m, index) {
        if (this.selectedHub && this.selectedHub.id === m.id) {
          return null;
        }
        return {
          color: 'white',
          fontSize: '16px',
          text: m.marker_title ?? '',
        }
      }
    }
  }
</script>
<style>
  .page-container {
  }

  .vue-map-container {
    position: unset !important;
  }

  .hub-list {
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    width: 300px;
    background: white;
    border-left: 1px solid gray;
    overflow-y: scroll;
  }

  @media only screen and (max-width: 600px) {
    .hub-list {
      left: 0;
      right: 0;
      bottom: 0;
      top: initial;
      height: 200px;
      width: initial;
      border-left: 0;
      border-top: 1px solid gray;
    }
  }

  .hub-item {
    display: flex;
    flex-direction: row;
    width: 100%;
    justify-content: space-between;
    align-items: center;
    padding-bottom: 10px;
    padding-top: 10px;
  }
</style>
