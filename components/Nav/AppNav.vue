<template>
  <div class="navigation-bar">
    <div class="navigation">
      <nav 
        role="navigation" 
        class="container is-flex navbar">
        <nuxt-link 
          to="/" 
          @click="refreshPage()" 
          class="logo col--8-mobile col--4-tablet is-center">
          <img 
            src="/images/logo.png" 
            alt="">
        </nuxt-link>
      
        <div 
          v-if="$store.state.window && $store.state.window < 1024"
          @click="$store.commit('openMenu')"
          class="menu menu--mobile"
        >
          <i class="fa fa-menu menu--mobile__burger"/>
        </div>
      
      
        <ul 
          v-else 
          class="menu menu--desktop">
          <nuxt-link
            :to="`/${menu}`"
            v-for="(menu, index) in menuItems"
            :key="index"
          >
            {{ menu === '' ? 'our phones' : menu }}
          </nuxt-link>

        </ul>
      </nav>
    </div>

    <no-ssr>
      <vue-media :query="{maxWidth: 1024}">
        <div 
          style="z-index: 9999" 
          class="modal-container is-hidden-desktop">
          <app-menu-mobile 
            :menu-items="menuItems"
          />
        </div>
      </vue-media>
    </no-ssr>
  </div>
  
</template>

<script>
  import debounce from "lodash/debounce";

  export default {
    name: "AppNav",
    data () {
      return {
        menuItems: ["", "about", "contact"]
      };
    },
    components: {
      AppMenuMobile: () => import("@/components/Nav/AppMenuMobile")
    },
    methods: {
      refreshPage () {
        if (this.$route.path === "/") {
          window.location.reload();
          window.scrollTo(0, 0);
        }
      }
    },
    async mounted () {
      if (process.browser) {
        // check connection type
        var connection =
          navigator.connection ||
          navigator.mozConnection ||
          navigator.webkitConnection;
        if (connection != null) {
          var type = connection.type;
          let vm = this;
          function updateConnectionStatus () {
            console.log(
              "Connection type changed from " + type + " to " + connection.type
            );
            vm.$store.commit("setConnection", connection.type);
          }
          connection.addEventListener("typechange", updateConnectionStatus);
        }
        // console.log("Connection type: " + type);
        this.$store.commit("setConnection", type);
        // on load trigger window width mutation once
        this.$store.commit("windowResize", window.innerWidth);

        window.onNuxtReady(app => {
          // resize triggers window width mutation
          window.addEventListener(
            "resize",
            debounce(() => {
              console.log("window resize");
              this.$store.commit("windowResize", window.innerWidth);
            }, 300)
          );
        });

        // route management for menus/state/transitions
        this.$root.$on("routeChanged", () => {
          // console.log('route changed, transitioning')
          this.$store.commit("resetMenus");
          let body = document.querySelector("body");
          let html = document.querySelector("html");
          if (body && html) {
            body.style.overflow = "visible";
            html.style.overflow = "visible";
            body.style.position = "static";
          }
        });
      }
    }
  };
</script>

<style lang="scss" scoped>
  .navigation {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 5;
    height: 60px;
    background-color: white;
    box-shadow: 0 0 2px 0 $grey;
    margin: 0 auto;
    .navbar {
      align-items: center;
      padding: 0 15px;
      height: 100%;
      justify-content: space-between;
      align-items: center;
      @media (min-width: $tablet) {
        padding: 0 36px;
      }
      img {
        width: 40px;
        height: 40px;
      }
      .menu {
        &--mobile {
          display: block;
        }
        &--desktop {
          display: flex;
          justify-content: space-around;
          a {
            cursor: pointer;
            font-size: 18px;
            text-transform: uppercase;
            font-weight: 400;
            transition: color 0.3s ease-in-out;
            &.nuxt-link-active,
            &:hover {
              font-weight: bold;
              color: $red;
            }
            &:not(:last-child) {
              margin-right: $gap;
            }
          }
        }
      }
    }
    i {
      color: $grey;
      cursor: pointer;
      font-size: 14px;
      @media (min-width: $tablet) {
        font-size: 16px;
      }
    }
  }
</style>
