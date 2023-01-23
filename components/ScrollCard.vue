<template>
  <div>
    <div
      class="mt-2 mb-4"
      v-for="(album, albumId) in groupedPhotos"
      :key="albumId"
    >
      <div class="d-flex justify-space-between">
        <h3>Album Id - {{ albumId }}</h3>
        <div>Show All</div>
      </div>
      <div class="scrolling-wrapper">
        <div class="d-flex mt-2">
          <v-card
            class="card px-4 pt-3 pb-n6 rounded-xl"
            outlined
            v-for="photo in album"
            :key="photo.id"
          >
            <v-img
              :src="photo.thumbnailUrl"
              class="photos"
              :alt="photo.title"
            />
            <h4 class="pt-2">{{ truncateTitle(photo.title) }}</h4>
            <a
              :href="photo.url"
              style="text-decoration: none; color: orange"
              class="pb-2"
              >View full size</a
            >
          </v-card>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    title: {
      type: String,
      default: ' ADD USER',
    },
  },
  data() {
    return {
      photos: [],
    }
  },
  computed: {
    groupedPhotos() {
      const groups = {}
      const photosArr = Object.values(this.photos)
      photosArr.forEach((photo) => {
        if (!groups[photo.albumId]) {
          groups[photo.albumId] = []
        }
        groups[photo.albumId].push(photo)
      })
      return groups
    },
  },
  mounted() {
    this.initialize()
  },
  methods: {
    initialize() {
      this.$axios
        .get('https://jsonplaceholder.typicode.com/photos')
        .then((res) => {
          debugger
          this.photos = { ...res.data }
        })
    },
    truncateTitle(title) {
      const words = title.split(' ')
      return words.slice(0, 2).join(' ')
    },
  },
}
</script>
<style lang="scss">
.scrolling-wrapper {
  display: flex;
  flex-wrap: nowrap;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}
.card {
  flex: 0 0 auto;
  width: 220px;
  height: auto;
  margin-right: 4rem;
  box-sizing: border-box;
}
.photos {
  height: 150px;
}

::-webkit-scrollbar {
  display: none;
}
</style>
