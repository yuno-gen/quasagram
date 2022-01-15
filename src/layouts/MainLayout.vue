<template>
  <q-layout view="lHh Lpr lFf">
    <q-header
      class="bg-white text-grey-10"
      bordered
      >
      <q-toolbar class="constrain">
        <q-btn
        to="/camera"
          flat
          round
          icon="eva-camera-outline q-rm-sm"
          size="18px"
          dense
          class="gt-xs"
        >

        <q-separator
          class="gt-xs"
          spaced
          vertical
        />

        </q-btn>
        <q-toolbar-title class="text-grand-hotel text-bold"> Quasagram </q-toolbar-title>

        <q-btn
          to="/"
          flat
          round
          icon="eva-home-outline"
          size="18px"
          dense
          class="gt-xs"
        />

      </q-toolbar>
    </q-header>

    <q-footer
      class="bg-white"
      bordered>
      <transition
        appear
        enter-active-class="animated backInLeft"
        leave-active-class="animated backOutLeft"
      >
        <div class="banner-container bg-primary" 
            v-if="showAppInstallBanner">
            
          <div class="constrain">
            <q-banner
              inline-actions
              dense
              class="bg-primary text-white text-bold">

              <template v-slot:avatar>
                <q-icon
                  name="eva-camera-outline"
                  color="white"
                  text-color="grey-10"
                  font-size="22px" />
              </template>

              Install Quasagram ?
              <template v-slot:action>
                <q-btn
                @click="installApp"
                  class="q-px-sm"
                  flat
                  label="Yes"
                  dense />
                <q-btn
                  @click="showAppInstallBanner = false"
                  class="q-px-sm"
                  flat
                  label="Later"
                  dense />
                <q-btn
                  @click="neverShowAppInstallBanner"
                  class="q-px-sm"
                  flat
                  label="Never"
                  dense />
              </template>
            </q-banner>
          </div>
          
        </div>
    </transition>
      

      <q-tabs
        active-color="primary"
        class="text-grey-10 lt-sm"
        indicator-color="transparent"
      >
       <q-route-tab
          to="/"
          icon="eva-home-outline" />
        <q-route-tab
          to="/camera"
          icon="eva-camera-outline"
           />
      </q-tabs>
    </q-footer>

    <q-page-container class="bg-grey-1">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
 let deferredPrompt;

export default {
  data() {
    return {
      showAppInstallBanner: false
    };
  },
  methods:{
    installApp(){
      this.showAppInstallBanner = false
      // Show the install prompt
      deferredPrompt.prompt();
      // Wait for the user to respond to the prompt
      deferredPrompt.userChoice.then((choiceResult)=>{
        if(choiceResult.outcome=='accepted'){
          console.log('User accepted the install prompt');
          this.neverShowAppInstallBanner()
        } else{
          console.log('User dismissed the install prompt');
        }
      })
    },
    neverShowAppInstallBanner(){
      this.showAppInstallBanner =  false
      console.log(this.showAppInstallBanner);
      this.$q.localStorage.set('neverShowAppInstallBanner', true)
    }
  },
  mounted(){
    let neverShowAppInstallBanner=this.$q.localStorage.getItem('neverShowAppInstallBanner')
    console.log(neverShowAppInstallBanner);
    if(!neverShowAppInstallBanner){
      window.addEventListener('beforeinstallprompt', (e) => {
      // Prevent the mini-infobar from appearing on mobile
      e.preventDefault();
      // Stash the event so it can be triggered later.
      deferredPrompt = e;
      // Update UI notify the user they can install the PWA
      setTimeout(() => {
        this.showAppInstallBanner = true
      }, 3000);
      
      // Optionally, send analytics event that PWA install promo was shown.
      console.log(`'beforeinstallprompt' event was fired.`);
    });
    }
    
  }
};
</script>

<style lang="sass">
.q-toolbar__title
    font-size: 30px
    @media (max-width: $breakpoint-xs-max)
      text-align: left
.q-footer
  .q-tab__icon
      font-size: 30px

.q-toolbar
  @media (min-width: $breakpoint-sm-min)
    height: 77px
</style>
