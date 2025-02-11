<template>
  <div class="case-study marketing" v-if="project != null">
    <WorkHero :project="project" :animateHeader="animateHeader"/>

    <ClientIntro :project="project" :animateIntro="animateIntro"/>

    <EmailNewsletter :project="project" :animateEmail="animateEmail"/>

    <DigitalInfographics :project="project" :animateDigital="animateDigital"/>

    <RichMedia :project="project" :animateRich="animateRich"/>

    <div
      class="work-navigation step"
      :class="{'animated': animateBottomImage}"
      v-if="previousProject && nextProject"
    >
      <div class="container-fluid is-flex">
        <nuxt-link class="previous" :to="`/${previousProject.slug}`">
          <LazyImage
            v-if="previousProject.acf.hero != null && previousProject.acf.status === 'true'"
            class="image"
            :hover="false"
            :image="previousProject.acf.hero.desktop_bg"
            :imageMobile="previousProject.acf.hero.mobile_bg"
          >
            <span>{{previousProject.acf.hero.title}}</span>
          </LazyImage>
          <p>Previous Project</p>
        </nuxt-link>
        <nuxt-link class="next" :to="`/${nextProject.slug}`">
          <LazyImage
            v-if="nextProject.acf.hero != null && nextProject.acf.status === 'true'"
            class="image"
            :hover="false"
            :image="nextProject.acf.hero.desktop_bg"
            :imageMobile="nextProject.acf.hero.mobile_bg"
          >
            <span>{{nextProject.acf.hero.title}}</span>
          </LazyImage>
          <p>Next Project</p>
        </nuxt-link>
      </div>
    </div>
  </div>
</template>

<script>
  import debounce from "lodash/debounce"
  import Config from "~/assets/config"
  import WorkHero from "@/components/Sections/Work/WorkHero"
  import LazyImage from "@/components/UI/LazyImage"
  import get from "lodash/get"
  let scroller, steps;

  export default {
    components: {
      WorkHero,
      LazyImage,
      ClientIntro: () => import("@/components/Sections/Work/ClientIntro"),
      EmailNewsletter: () => import("@/components/Sections/Work/EmailNewsletter"),
      DigitalInfographics: () =>
        import("@/components/Sections/Work/DigitalInfographics"),
      RichMedia: () => import("@/components/Sections/Work/RichMedia")
    },
    head () {
      if (
        this.project &&
        this.projectTitle != null &&
        this.project.seo != null &&
        this.project.seo.facebook.sizes != null &&
        this.project.seo.twitter.sizes != null
      ) {
        return {
          title: this.capitalizeFirstLetter(this.projectTitle.replace(/-/g, " ")),
          meta: [
            {
              hid: "description",
              name: "description",
              content: this.project.seo.description
            },
            {
              hid: "keywords",
              name: "keywords",
              content:
                this.project.keywords ||
                `${this.projectTitle.replace(
                  /-/g,
                  " "
                )}, ${this.projectTitle.replace(/-/g, ", ")}`
            },
            {
              hid: "description",
              itemprop: "description",
              content: this.project.seo.description
            },
            {
              hid: "image",
              itemprop: "image",
              content: this.project.seo.facebook.sizes.large
            },
            {
              hid: "twitter:title",
              name: "twitter:title",
              content: this.capitalizeFirstLetter(
                this.projectTitle.replace(/-/g, " ")
              )
            },
            {
              hid: "twitter:description",
              name: "twitter:description",
              content: this.project.seo.description
            },
            {
              hid: "twitter:image",
              name: "twitter:image",
              content: this.project.seo.twitter.sizes.large
            },
            {
              hid: "og:title",
              property: "og:title",
              content: this.capitalizeFirstLetter(
                this.projectTitle.replace(/-/g, " ")
              )
            },
            { hid: "og:url", property: "og:url", content: this.$route.path },
            {
              hid: "og:description",
              property: "og:description",
              content: this.project.seo.description
            },
            {
              hid: "og:image",
              property: "og:image",
              content: this.project.seo.facebook.sizes.large
            }
          ]
        };
      } else {
        return {
          title: "Case Study"
        };
      }
    },
    data () {
      return {
        animateHeader: false,
        animateIntro: false,
        animateEmail: false,
        animateDigital: false,
        animateRich: false,
        animateBottomImage: false
      };
    },
    async created () {
      if (!this.$store.state.projects) {
        const { data } = await this.$http.$get(Config.wpDomain + Config.api.projects)
        this.$store.commit("setProjects", data)
      }
    },
    async mounted () {
      if (process.browser) {
        setTimeout(() => {
          this.animateHeader = true;
          this.handleScroll();
        }, 150);
      }
    },
    methods: {
      hideMenu () {
        this.$store.commit("hideMenuBg");
      },
      showMenu (response) {
        if (response.index === 0) {
          this.$store.dispatch("showMenu");
        }
      },
      handleStepEnter (response) {
        // console.log(response.element);
        response.element.classList.add("animated");
        const length = document.querySelectorAll(".step").length - 2;
        switch (response.index) {
          case 0:
            this.hideMenu();
            this.animateHeader = true;
            break;
          case 1:
            this.animateIntro = true;
            break;
          case 2:
            this.animateEmail = true;
            break;
          case 3:
            this.animateDigital = true;
            break;
          case 3:
            this.animateRich = true;
            break;
          case length:
            this.animateBottomImage = true;
            break;
          default:
            break;
        }
      },
      handleScroll () {
        if (window.innerWidth > 577) {
          scroller = this.scrollama();
          steps = null;
          steps = scroller
            .setup({
              step: ".step",
              offset: 0.8,
              debug: false
            })
            .onStepEnter(this.handleStepEnter)
            .onStepExit(this.showMenu);

          steps.resize();
          steps.enable();
        } else {
          scroller = this.scrollama();
          steps = null;
          steps = scroller
            .setup({
              step: ".step",
              offset: 0.9,
              debug: false
            })
            .onStepEnter(this.handleStepEnter)
            .onStepExit(this.showMenu);

          steps.resize();
          steps.enable();
        }

        window.addEventListener(
          "resize",
          this.scrollamaResize,
          { passive: true },
          false
        );
      },
      scrollamaResize: debounce(function () {
        let step = document.querySelector(".step");
        if (step && step.length) {
          this.handleScroll();
        }
      }, 150)
    },
    computed: {
      projects () {
        if (!this.$store.state.projects.length) return false;
        return this.$store.state.projects;
      },
      projectTitle () {
        if (
          !this.$store.state.projects.length ||
          !this.$store.state.projects[this.getIndex]
        ) {
          return "";
        }
        return this.$store.state.projects[this.getIndex].slug;
      },
      project () {
        if (
          !this.$store.state.projects.length ||
          !this.$store.state.projects[this.getIndex]
        ) {
          return false;
        }
        return get(this.$store.state.projects[this.getIndex], "acf", false);
      },
      getIndex () {
        if (!this.projects.length) return 0;
        return this.projects.findIndex(index => {
          if (this.$route.params.work) {
            return this.$route.params.work === index.slug;
          }
          return this.$route.name === index.slug;
        });
      },
      previousProject () {
        if (this.getIndex === 0) {
          return this.projects[this.projects.length - 1];
        } else return this.projects[this.getIndex - 1];
      },
      nextProject () {
        if (this.getIndex === this.projects.length - 1) {
          return this.projects[0];
        } else return this.projects[this.getIndex + 1];
      },
      bgImage () {
        if (process.browser) {
          if (this.$store.state.window < 577) {
            return get(
              this.project,
              "hero.mobile_bg.sizes.large",
              "https://placehold.it/2048/2048"
            );
          } else if (
            this.$store.state.window > 576 &&
            this.$store.state.window < 1440
          ) {
            return get(
              this.project,
              "hero.desktop_bg.sizes.large",
              "https://placehold.it/2048/2048"
            );
          } else
            return get(
              this.project,
              "hero.desktop_bg.sizes.ultra",
              "https://placehold.it/2048/2048"
            );
        }
        return get(
          this.project,
          "hero.desktop_bg.sizes.large",
          "https://placehold.it/2048/2048"
        );
      }
    },
    beforeDestroy () {
      if (typeof scroller !== "undefined") {
        scroller.disable && scroller.disable();
      }
      scroller = null;
      steps = null;
      window.removeEventListener("resize", this.scrollamaResize, false);
    }
  };
</script>

<style lang='scss' scoped>
  h1 {
    padding-bottom: $gap;

    @include media(sm) {
      padding-bottom: 60px;
      max-width: 280px;
    }
  }
  section {
    margin: $gap * 1.5 $gap;

    @include media(md) {
      margin: $gap * 3 auto;
    }
  }

  .case-study {
    &.marketing {
      .client-intro {
        margin-bottom: 0;
      }
    }
  }

  .work-navigation {
    margin: $gap * 2 0 $gap * 1.5;
    height: 100%;
    @include fadeInUp;
    transition-delay: 1s;

    p {
      font-size: $font-size;
    }

    .container-fluid {
      justify-content: space-between;
      padding: 0 $gap;
      height: 160px;
      position: relative;
      overflow: hidden;

      @include media(md) {
        padding: 0;
        margin-left: auto;
      }

      @supports (display: grid) {
        height: 100%;
      }
    }

    /deep/ img {
      padding-top: 0 !important;
    }

    a {
      position: relative;
      bottom: 0;
      height: 100%;
      transition: transform 0.4s ease-in-out;

      &:after {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 1;
        background: rgba(0, 0, 0, 0.4);
        opacity: 0;
        transition: opacity 0.2s ease-in-out;

        @supports (display: grid) {
          bottom: 20px;
        }
      }

      p {
        font-weight: bold;
        color: black;
        text-decoration: none;
      }

      span {
        @include size(h4);
        color: white;
        max-width: 100px;
        position: absolute;
        pointer-events: none;
        top: 50%;
        left: 50%;
        opacity: 0;
        z-index: 2;
        transition: all 0.4s ease-in-out;
      }

      &.previous {
        text-align: right;
        @include media(md) {
          transform: translateX(-120px);
        }

        span {
          @include media(md) {
            transform: translate(-100%, -50%);
          }
        }
      }

      &.next {
        @include media(md) {
          transform: translateX(120px);
        }

        span {
          @include media(md) {
            transform: translate(100%, -50%);
          }
        }
      }

      @include media(md) {
        &:hover {
          &.next,
          &.previous {
            transform: translateX(0);
          }
          span {
            opacity: 1;
            transform: translate(-50%, -50%);
          }
          &:after {
            opacity: 1;
          }
        }
      }

      /deep/ img {
        display: none;

        @include media(md) {
          display: block;
          width: 250px;
          height: 180px;
          object-fit: cover;
          object-position: center;
          position: relative;
          bottom: 20px;
          left: 0;
        }
      }
    }
  }
</style>
