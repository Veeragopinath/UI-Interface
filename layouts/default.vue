<template>
  <v-app class="grey lighten-5" id="inspire">
    <v-row class="grey lighten-5 d-flex mx-10 align-center my-2" app>
      <div col="3" class="mr-12">
        <v-img
          max-height="75"
          max-width="100"
          src="https://www.wattawow.com/_nuxt/img/Gallery_logo.85d34f3.png"
        ></v-img>
      </div>
      <v-col
        class="d-flex font-weight-medium align-center space-between pb-0 ml-8"
        col="7"
      >
        <a class="mr-4 text-decoration-underline black--text mr-12">Home</a>
        <div class="mr-4">
          <v-dialog v-model="dialog" width="500">
            <template v-slot:activator="{ on, attrs }">
              <a class="black--text" elevation="0" v-bind="attrs" v-on="on">
                About Us
              </a>
            </template>

            <AboutUs></AboutUs>
          </v-dialog>
        </div>
      </v-col>
      <v-spacer></v-spacer>
      <v-btn class="indigo darken-4 white--text"> Sign In </v-btn>
    </v-row>

    <v-row class="grey lighten-5 mx-10 my-3 mt-6 d-flex justify-space-between">
      <div col="3">Hello <span class="font-weight-black"> Champ,</span></div>

      <div col="4"></div>

      <v-chip class="ml-16" outlined> #LifeSavingSticker</v-chip>
      <div>
        <v-icon class="mdi-map-marker-plus"></v-icon>
        <span>Chennai</span>
      </div>
    </v-row>

    <v-main class="grey lighten-5 mx-10">
      <nuxt />
    </v-main>
  </v-app>
</template>

<script>
export default {
  data: () => ({}),
}
</script>
<style>
.v-application {
  background-color: #fafafa !important;
}
.gap-between {
  margin-left: 10px !important;
}
</style>
