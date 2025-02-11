<template>
  <div class="contact">
    <section class="section hero">
      <div class="grid-wrapper" v-if="contactPage != null">
        <lazy-image
          class='image'
          v-if="contactPage.acf"
          noBg
          :image="contactPage.acf.background"
          :title="contactPage.acf.title"
          positionMobile="left"
          :hover="false"
          :imageMobile="contactPage.acf.background"
          home
        />
        <div class="container">
          <h1 class="jumbo" v-if="contactPage.acf">
            {{contactPage.acf.title}}
          </h1>
          <h3>Email:
            <a href="mailto:hello@nunziellasalluce.com?subject=hello">
              hello@nunziellasalluce.com
            </a>
          </h3>
        </div>

        <div class="contact-form--wrapper" :class="{'animated': $root.showForm}">
          <the-contact-form />
        </div>
      </div>
    </section>
  </div>
</template>

<script>
  import TheContactForm from '@/components/UI/TheContactForm'
  import LazyImage from "@/components/UI/LazyImage"
  import Config from '~/assets/config.js'

  export default {
    async asyncData ({ $http }) {
      const data = await $http.$get(Config.wpDomain + Config.api.contactPage)
      return { contactPage: data }
    },
    components: {
      TheContactForm,
      LazyImage
    },
    mounted () {
      if (process.browser) {
        setTimeout(() => {
          this.$root.showForm = true
        }, 1000)
      }
    }
  }
</script>

<style lang="scss" scoped>
  .contact {
    section {
      margin: $gap * 1.5 $gap;

      @include media(md) {
        margin: $gap * 3 auto;
      }

      &.hero {
        background: white;
        margin: 0;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        justify-content: center;
        position: relative;
        padding: 0;

        @include media(md) {
          margin: $gap * 3 0;
          padding-top: 90px;
        }

        .grid-wrapper {
          min-height: 100vh;
          width: 100%;
          grid-template-rows: repeat(12, auto);
          grid-template-columns: 1fr;

          @include media(xxl) {
            min-height: 70vh;
          }
        }

        .container {
          display: flex;
          width: 100%;
          flex-direction: column;
          grid-column: 1 / -1;
          grid-row: 4 / 5;
          padding: $gap;

          @include media(sm) {
            padding: 0;
          }
        }

        .image {
          grid-column: 1 / 2;
          width: 100%;
          z-index: 0;
          grid-row: 1 / 11;

          @include media(xxl) {
            grid-row: 1 / 11;
            max-height: 80vh;
          }
        }

        h1,
        h3 {
          color: black;
          grid-column: 1 / 2;
          z-index: 1;
          padding: 0;
        }

        h1 {
          max-width: 150px;
          padding-top: 60px;
          @include media(sm) {
            max-width: 350px;
            padding-top: 96px;
          }
          grid-row: 2 / 4;
        }

        h3 {
          grid-row: 4 / 7;
        }

        .contact-form--wrapper {
          grid-column: 1 / 2;
          z-index: 1;
          grid-row: 8 / -1;
          padding-top: 50px;
          padding: 0;
          max-width: 100%;
          overflow: hidden;

          @include media(lg) {
            margin: 0;
            padding: 0 $gap;
            margin-left: auto;
            margin-right: 10vw;
          }
        }

        @include media(sm) {
          padding: 0;
        }

        @include media(md) {
          margin: 0 auto $gap * 3;
        }

        .container {
          height: 100%;
          justify-content: center;
        }

        a {
          color: black;
          font-weight: 600;
          text-decoration: none;

          &:hover {
            text-decoration: underline;
          }
        }
      }
      .contact-form--wrapper {
        @include media(lg) {
          margin-left: auto;
          margin-right: 10vw;
          min-height: 546px;
        }
        @include fadeInUp;

        &.animated {
          opacity: 1;
        }
      }
    }
  }
</style>
