<template>
  <div id="app">
    <!-- Navbar -->
    <navbar></navbar>
    <div class="container">
      <flash-message class="navbar-fixed-top"></flash-message>
      <modal></modal>
      <modalForm></modalForm>
      <!-- Main page content -->
      <router-view/>
    </div>
  </div>
</template>

<script>

// LiveKitchen connection via WebSockets
import Vue from 'vue';
import VueSocketio from 'vue-socket.io';
import settings from '../config/settings';
import roles from './mixins/userRoles';

// Global components
import Navbar from './components/Navbar';
import Modal from './components/Modal';
import ModalForm from './components/ModalForm';

export default {
  name: 'app',
  components: {
    'navbar': Navbar,
    'modal': Modal,
    'modalForm': ModalForm,
  },

  created() {
    if(!this.userIsAuthenticated) {
      // If the user is not logged in, redirect them to the home page when they visit any other page
      if(this.$route.path != '/') {
        this.$router.push('/');
      }
    }

    if(localStorage.getItem('restaurant') !== null) {
      const r = JSON.parse(localStorage.restaurant);
      if(r.hasOwnProperty('restaurantId')) {
        const userObj = JSON.stringify({ role: roles.restaurateur, id: r.restaurantId });
        const query = '?user='.concat(userObj);
        const route = settings.webSocketsUrl.concat(query);
        Vue.use(VueSocketio, route);
      }
    }

    // Redirect user if route is invalid
    for(var route of this.$router.options.routes) {
      if(this.$route.path == route.path) return;
    }
    this.$router.push('/');
    
  },

  computed: {
    userIsAuthenticated() {
      return this.$store.getters.isUserAuthenticated;
    }
  },

  watch: {
    '$route' (to, from) {
      this.flash().destroyAll();
      if(to.hasOwnProperty('query')) {
        if(to.query.hasOwnProperty('logout')) {
          if(to.query.logout == true) {
            // reset the state
            location.reload();
          }
        }
      }
    }
  }
}
</script>

<style>

  #app {
    font-family: Helvetica, 'Avenir', Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    /*color: #2c3e50;*/
    margin-top: 50px;
  }

  /** 
    Cheating a bit, but when building the app (npm run build), 
    the two stylings below are not adhered to. There must be a conflict
    somewhere in the CSS, which manifests following minification. 

    But I can't be bothered to find it, and I was under the impression
    that CSS properties set here would be respected globally (so long as 'scoped' was removed from
    the style tag).
  **/

  body {
    background-color: #1b1c23 !important;
  }

  .container-fluid {
    padding: 0 !important;
  }

  .flash__message.error{
    background-color: #e60000 !important;
    border-color: #e60000 !important;
    color: white !important;
    font-size: 12px !important;
  }

  .flash__message.success{
    background-color: #469ada !important;
    border-color: #469ada !important;
    color: white !important;
    font-size: 12px !important;
  }

</style>
