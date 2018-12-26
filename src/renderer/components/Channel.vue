<template>
  <div>
    <p><img :src='channel.thumbnail' class="image-round"/>{{ channel.title }}</p>
    <router-link :to="{ name: 'home' }"> back </router-link>
    <el-button @click="getVideos(channel.id)" type="primary">動画リスト取得</el-button>
    <br/>
    <div class="ytb-row" v-for="video in videos">
      <div class="ytb-thumbnail">
        <img :src="video.snippet.thumbnails.medium.url"/>
      </div>
      <div class="ytb-detail">
        <el-button @click="watchVideo(video.id.videoId)" type="primary">WATCH</el-button>
        <p>{{ video.snippet.title }}</p>
        <p>{{ video.snippet.description }}</p>
        <p>{{ video.snippet.publishedAt }}</p>
      </div>
    </div>
    <br/>
  </div>
</template>

<script>
const {google} = require('googleapis')
const youtube = google.youtube({
  version: 'v3',
  auth: localStorage.getItem('authKey')
})
const {shell} = require('electron')

export default {
  data () {
    return {
      channel: {
        id: '',
        title: '',
        thumbnail: ''
      },
      videos: []
    }
  },
  methods: {
    watchVideo: function (videoId) {
      shell.openExternal('https://www.youtube.com/watch?v=' + videoId)
    },
    getVideos: function (channelId) {
      this.searchVideos(channelId, null)
    },
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
    },
    searchVideos: function (channelId, nextPageToken) {
      const params = {
        part: 'snippet',
        channelId: channelId,
        maxResults: 50,
        order: 'date'
      }
      if (nextPageToken) {
        params['nextPageToken'] = nextPageToken
      }

      youtube.search.list(params, (err, data) => {
        if (err) {
          console.log(err)
        }
        this.videos = data.data.items
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
.ytb-row {
  display: flex;
  margin-bottom: 10px;
}
.ytb-thumbnail {
  display: inline-block;
  position: relative;
  padding-right: 20px;
}
</style>
