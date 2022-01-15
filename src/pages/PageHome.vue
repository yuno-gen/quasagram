<template>
  <q-page class="constrain q-pa-md">
    <transition
      appear
      enter-active-class="animated backInLeft"
      leave-active-class="animated backOutLeft"
    >
      <div
        class="banner-container bg-primary"
        v-if="showNotificationBanner && pushNotificationSupported"
      >
        <div class="constrain">
          <q-banner class="bg-grey-3 q-mb-md">
            <template v-slot:avatar>
              <q-icon
                name="eva-bell-outline"
                color="primary"
                text-color="grey-10"
                font-size="22px"
              />
            </template>

            Enable Notification?
            <template v-slot:action>
              <q-btn
                @click="enableNotification"
                class="q-px-sm"
                flat
                color="primary"
                label="Yes"
                dense
              />
              <q-btn
                @click="showNotificationBanner = false"
                class="q-px-sm"
                flat
                color="primary"
                label="Later"
                dense
              />
              <q-btn
                @click="neverShowNotificationBanner"
                class="q-px-sm"
                flat
                color="primary"
                label="Never"
                dense
              />
            </template>
          </q-banner>
        </div>
      </div>
    </transition>

    <div class="row q-col-gutter-lg">
      <div class="col-12 col-sm-8">
        <template v-if="!loadingPosts && posts.length">
          <q-card
            v-for="post in posts"
            :key="post.id"
            class="card-post q-mb-md"
            flat
            bordered
          >
            <q-item class="gt-sm">
              <q-item-section avatar>
                <q-avatar>
                  <img src="avatar.jpg" />
                </q-avatar>
              </q-item-section>

              <q-item-section>
                <q-item-label class="text-bold">a_pande</q-item-label>
                <q-item-label caption> {{ post.location }} </q-item-label>
              </q-item-section>
            </q-item>

            <q-separator />

            <q-img :src="post.imageUrl" />

            <q-card-section>
              <div class="text-h6">{{ post.caption }}</div>
              <div class="text-caption text-grey">
                {{ post.date | niceDate }}
              </div>
            </q-card-section>
          </q-card>
        </template>

        <template v-else-if="!loadingPosts && !posts.length">
          <h5 class="text-center text-grey">No posts yet</h5>
        </template>

        <template v-else>
          <q-card flat bordered>
            <q-item>
              <q-item-section avatar>
                <q-skeleton type="QAvatar" animation="fade" size="40px" />
              </q-item-section>

              <q-item-section>
                <q-item-label>
                  <q-skeleton type="text" animation="fade" />
                </q-item-label>
                <q-item-label caption>
                  <q-skeleton type="text" animation="fade" />
                </q-item-label>
              </q-item-section>
            </q-item>

            <q-skeleton height="200px" square animation="fade" />

            <q-card-section>
              <q-skeleton type="text" class="text-subtitle2" animation="fade" />
              <q-skeleton
                type="text"
                width="50%"
                class="text-subtitle2"
                animation="fade"
              />
            </q-card-section>
          </q-card>
        </template>
      </div>
      <div class="col-4 gt-sm">
        <q-item class="fixed">
          <q-item-section avatar>
            <q-avatar size="48px">
              <img src="avatar.jpg" />
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-bold">a_pande</q-item-label>
            <q-item-label caption> a_pande </q-item-label>
          </q-item-section>
        </q-item>
      </div>
    </div>
  </q-page>
</template>

<script>
import { date } from "quasar";

export default {
  name: "PageHome",
  data() {
    return {
      posts: [],
      loadingPosts: false,
      showNotificationBanner: false,
    };
  },
  computed: {
    pushNotificationSupported() {
      if ("PushManager" in window) return true;
      return false;
    },
  },
  methods: {
    getPosts() {
      this.loadingPosts = true;
      this.$axios
        .get(`${process.env.API}/posts`)
        .then((response) => {
          console.log(response);
          this.posts = response.data;
          this.loadingPosts = false;
        })
        .catch((err) => {
          this.$q.dialog({
            title: "Error",
            message: "Could not download your posts",
          });
          this.loadingPosts = false;
        });
    },
    initNotificationBanner() {
      let neverShowNotificatinBanner = this.$q.localStorage.getItem(
        "neverShowNotificationBanner"
      );

      if (!neverShowNotificatinBanner) {
        this.showNotificationBanner = true;
      }
    },
    enableNotification() {
      if (this.pushNotificationSupported) {
        Notification.requestPermission((result) => {
          console.log("result: ", result);

          if (result == "granted") {
            this.displayGrantedNotification();
          }
          this.neverShowNotificationBanner();
        });
      }
    },
    displayGrantedNotification() {
      new Notification("You are subscribed to notification", {
        body: "Thank you for subscribing",
        icon: "icons/icon-128x128.png",
        image: "icons/icon-128x128.png",
        badge: "icons/icon-128x128.png",
        // dir: 'ltr'
        lang: "en-us",
        vibrate: [100, 50, 200],
        tag: "confirm-notification",
        renotify: true,
        // actions:[
        //   {
        //     action: 'hello',
        //     title: 'Hello',
        //     icon: 'icons/icon-128x128.png'
        //   },
        //   {
        //     action: 'goodbye',
        //     title: 'GoodBye',
        //     icon: 'icons/icon-128x128.png'
        //   }
        // ]
      });
    },
    neverShowNotificationBanner() {
      this.showNotificationBanner = false;
      console.log(this.showNotificationBanner);
      this.$q.localStorage.set("neverShowNotificationBanner", true);
    },
  },
  filters: {
    niceDate(value) {
      return date.formatDate(value, "MMMM D h:mmA");
    },
  },
  created() {
    this.getPosts();
    this.initNotificationBanner();
  },
};
</script>

<style lang="sass">
.card-post
  .q-img
    min-height: 200px
</style>