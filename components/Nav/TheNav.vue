<template>
  <div class="navigation-bar">
    <div
      class="navigation"
      :class="{'scrolled': $store.state.menuScrolled, 'done': $store.state.menuScrolled && $store.state.menuScrolledDone || $route.path === '/about' || $route.path === '/contact' || $route.path === '/privacy-policy', 'about': $route.path === '/about', 'contact': $route.path === '/contact', 'static': $route.path === '/privacy-policy'}"
    >
      <nav role="navigation" class="container is-flex navbar">
        <nuxt-link
          to="/"
          no-prefetch
          @mouseover.native="animating = true"
          @mouseleave.native="animating = false"
          class="logo col--8-mobile col--4-tablet is-center"
        >
          <TheLogoStatic
            :width="90"
            :height="46"
            :mobileWidth="50"
            :mobileHeight="31"
            :animating="animating"
            :fill="$store.state.menuScrolled ? '#f4a261' : 'white'"
          />
          <TheLogo
            :width="90"
            :height="46"
            :mobileWidth="50"
            :mobileHeight="31"
            :animating="animating"
            :fill="$store.state.menuScrolled ? '#f4a261' : 'white'"
          />
        </nuxt-link>

        <div
          v-if="$store.state.window && $store.state.window < 1024"
          @click="$store.commit('openMenu')"
          class="menu menu--mobile"
        >
          <transition-group name="rotate" mode="out-in">
            <div class="rotate" key="closed" v-if="!$store.state.navOpen">
              <BurgerMenu
                :fill="$route.path === '/' && 'black' || $store.state.menuScrolled ? 'black' : 'white'"
                :stroke="$route.path === '/' && 'black' || $store.state.menuScrolled ? 'black' : 'white'"
              />
            </div>
            <div class="rotate" key="open" v-else>
              <div :class="$route.path === '/' && 'black'" class="close-icon">
                <span class="close-icon--line"/>
                <span class="close-icon--line inverted"/>
              </div>
            </div>
          </transition-group>
        </div>

        <ul
          v-else
          class="menu menu--desktop"
          :class="($route.path === '/' || $route.path === '/contact') && 'black'"
        >
          <nuxt-link
            no-prefetch
            to="/"
            :class="$route.path === '/' && $route.hash !== '#work' && 'nuxt-link-active'"
            exact
          >Home</nuxt-link>
          <nuxt-link
            no-prefetch
            :class="$route.hash === '#work'
              && 'nuxt-link-active'"
            to="/#work"
            exact
            v-scroll-to="{element:'.projects'}"
          >Work</nuxt-link>
          <nuxt-link no-prefetch to="/about" exact>About</nuxt-link>
          <nuxt-link no-prefetch to="/contact" exact>Contact</nuxt-link>
        </ul>
      </nav>
    </div>

    <no-ssr>
      <vue-media :query="{maxWidth: 1024}">
        <div style="z-index: 9999" class="modal-container is-hidden-desktop">
          <TheMenuMobile :menu-items="menuItems"/>
        </div>
      </vue-media>
    </no-ssr>
  </div>
</template>

<script>
  import debounce from "lodash/debounce";
  import TheLogo from "@/components/Icons/TheLogo";
  import TheLogoStatic from "@/components/Icons/TheLogoStatic";
  import BurgerMenu from "@/components/Icons/BurgerMenu";

  export default {
    name: "TheNav",
    data () {
      return {
        menuItems: ["", "work", "about", "contact"],
        animating: false
      };
    },
    components: {
      TheMenuMobile: () => import("@/components/Nav/TheMenuMobile"),
      TheLogo,
      TheLogoStatic,
      BurgerMenu
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
            vm.$store.commit("setConnection", connection.type);
          }
          connection.addEventListener("typechange", updateConnectionStatus);
        }
        this.$store.commit("setConnection", type);
        // on load trigger window width mutation once
        this.$store.commit("windowResize", window.innerWidth);

        window.onNuxtReady(app => {
          // resize triggers window width mutation
          window.addEventListener(
            "resize",
            debounce(() => {
              this.$store.commit("windowResize", window.innerWidth);
            }, 300)
          );
        });

        // route management for menus/state/transitions
        this.$root.$on("routeChanged", () => {
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
    top: 0;
    left: 0;
    width: 100%;
    z-index: 5;
    height: 60px;
    background-color: transparent;
    box-shadow: unset;
    margin: 0 auto;
    padding: 0 $gap;
    position: fixed;

    @include media(sm) {
      height: 90px;
      padding: 0;
    }

    @include media(lg) {
      position: absolute;
    }

    .navbar {
      align-items: center;
      height: 100%;
      justify-content: space-between;
      align-items: center;

      .logo {
        &:before,
        &:after {
          display: none;
        }
      }
      img {
        width: 50px;
        height: 31px;
        object-fit: contain;
        stroke: $primary;
      }
      .menu {
        cursor: pointer;
        .rotate {
          cursor: pointer;
        }
        &--mobile {
          display: block;
          position: relative;
        }
        &--desktop {
          display: flex;
          justify-content: space-around;

          a {
            cursor: pointer;
            font-size: $font-size + 4px;
            text-transform: capitalize;
            font-weight: 400;
            color: white;
            transition: color 0.3s ease-in-out;

            &.nuxt-link-active {
              color: $primary;
            }

            &:not(:last-child) {
              margin-right: $gap;
            }

            position: relative;
            padding: $gap / 3 0;

            &:hover {
              color: #fff;
              text-decoration: none;
            }

            &:before,
            &:after {
              content: "";
              position: absolute;
              width: 0%;
              height: 2px;
              bottom: -2px;
              background: #fff;
            }

            &:before {
              left: 0;
            }

            &:after {
              right: 0;
              background: #fff;
              transition: width 0.8s cubic-bezier(0.22, 0.61, 0.36, 1);
            }

            &:hover:before {
              background: #fff;
              width: 100%;
              transition: width 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
            }

            &:hover:after {
              background: transparent;
              width: 100%;
              transition: 0s;
            }

            &.nuxt-link-active {
              color: $primary;

              &:before,
              &:after {
                background: $primary;
              }

              &:after {
                right: 0;
                background: $primary;
              }

              &:hover {
                &:before {
                  background: $primary;
                }
                &:after {
                  background: transparent;
                  width: 100%;
                  transition: 0s;
                }
              }
            }
          }

          &.black {
            a:not(.nuxt-link-active) {
              color: black;

              &:hover {
                color: black;
                text-decoration: none;
              }

              &:before,
              &:after {
                content: "";
                position: absolute;
                width: 0%;
                height: 2px;
                bottom: -2px;
                background: black;
              }

              &:before {
                left: 0;
              }

              &:after {
                right: 0;
                background: black;
                transition: width 0.8s cubic-bezier(0.22, 0.61, 0.36, 1);
              }

              &:hover:before {
                background: black;
                width: 100%;
                transition: width 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
              }
            }
          }
        }
      }
    }

    &.scrolled {
      @include media(lg) {
        transform: translateY(-100%);
      }

      .navbar a {
        color: $secondary;

        &:before,
        &:after {
          background: $primary;
        }

        &:hover:before {
          background: $primary;
          width: 100%;
          transition: width 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
        }

        &:hover:after {
          background: transparent;
          width: 100%;
          transition: 0s;
        }
      }

      .close-icon {
        cursor: pointer;
        &--line {
          background: black;
        }
      }
    }

    &.scrolled.done {
      @include media(lg) {
        position: fixed;
        background-color: $lightgrey;
        transition: all 0.6s ease-in-out;
        transform: translateY(0);
      }
    }

    &.contact,
    &.about,
    &.scrolled.done.contact,
    &.scrolled.done.about {
      background-color: transparent;
      .navbar a {
        color: white;
      }
    }

    &.static {
      background-color: $grey;
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
  .close-icon {
    cursor: pointer;
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;

    &--line {
      position: absolute;
      top: 50%;
      cursor: pointer;
      display: block;
      margin-bottom: 3px;
      background: white;
      border-radius: 2px;
      opacity: 1;
      height: 2px;
      width: 15px;
      transform: rotate(45deg);
      &.inverted {
        transform: rotate(-45deg);
      }
    }

    &.black .close-icon--line {
      background: black;
    }
  }
</style>
