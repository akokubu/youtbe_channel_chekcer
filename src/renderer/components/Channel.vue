<template>
  <div>
    <p><img :src='channel.thumbnail' class="image-round"/>{{ channel.title }}</p>
    <router-link :to="{ name: 'home' }"> back </router-link>
  </div>
</template>

<script>
const {google} = require('googleapis')
const youtube = google.youtube({
  version: 'v3',
  auth: localStorage.getItem('authKey')
})

export default {
  data () {
    return {
      channel: {
        id: '',
        title: '',
        thumbnail: ''
      }
    }
  },
  methods: {
    getChannel (channelId) {
      const params = {
        part: 'snippet',
        id: channelId
      }

      youtube.channels.list(params, (err, data) => {
        if (err) {
          console.log(err)
        }
        var ch = data.data.items[0].snippet
        this.channel = {
          id: params.id,
          title: ch.title,
          thumbnail: ch.thumbnails.medium.url
        }
      })
    }
  },
  mounted: function () {
    this.getChannel(this.$route.params.id)
  }
}
</script>

<style>
.image-round {
  border-radius: 50%;
}
</style>
