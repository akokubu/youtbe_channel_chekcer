<template>
  <div>
    <p><img :src='channel.thumbnail' class="image-round"/>{{ channel.title }}</p>
    <router-link :to="{ name: 'home' }"> back </router-link>
    <el-button @click="getVideos(channel.id)" type="primary">動画リスト取得</el-button>
    <br/>
    <div v-for="video in videos">
      <yt-video :video="video" v-on:video-downloaded="videoDownloaded"></yt-video>
    </div>
    <br/>
  </div>
</template>

<script>
import path from 'path'
import { remote } from 'electron'
import YtVideo from './Video.vue'

const {google} = require('googleapis')
const youtube = google.youtube({
  version: 'v3',
  auth: localStorage.getItem('authKey')
})

var Datastore = require('nedb')
var db = new Datastore({
  filename: path.join(remote.app.getPath('userData'), '/video.db'),
  autoload: true
})

export default {
  components: {
    YtVideo
  },
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
    getVideos: function (channelId) {
      var self = this
      db.find({channelId: channelId}, function (err, results) {
        if (err) {
          console.log(err)
        }
        var lastUpdate = null
        if (results.length > 0) {
          results.sort(function (a, b) {
            if (a.publishedAt > b.publishedAt) return -1
            if (a.publishedAt < b.publishedAt) return 1
            return 0
          })
          self.videos = results

          var dt = new Date(results[0].publishedAt)
          dt.setSeconds(dt.getSeconds() + 1)
          lastUpdate = dt.toISOString().split('.')[0] + 'Z'
        }
        self.searchVideos(channelId, lastUpdate, null, function (videos) {
          if (videos.length > 0) {
            db.insert(videos, function (err, newvideos) {
              if (err) {
                console.log(err)
              }
              Array.prototype.push.apply(newvideos, results)
              self.videos = newvideos
            })
          }
        })
      })
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
    searchVideos: function (channelId, lastUpdate, nextPageToken, callback) {
      const params = {
        part: 'snippet',
        channelId: channelId,
        maxResults: 50,
        type: 'video',
        order: 'date'
      }
      if (nextPageToken) {
        params['pageToken'] = nextPageToken
      }
      if (lastUpdate) {
        params['publishedAfter'] = lastUpdate
      }
      var self = this
      youtube.search.list(params, (err, data) => {
        if (err) {
          console.log(err)
        }
        var list = []
        data.data.items.forEach(function (video) {
          list.push({
            channelId: channelId,
            id: video.id.videoId,
            title: video.snippet.title,
            despcription: video.snippet.description,
            publishedAt: video.snippet.publishedAt,
            thumbnail: video.snippet.thumbnails.medium.url
          })
        })
        var next = data.data.nextPageToken
        if (!next) {
          callback(list)
          return
        }

        self.searchVideos(channelId, lastUpdate, next, function (nextList) {
          Array.prototype.push.apply(list, nextList)
          callback(list)
        })
      })
    },
    videoDownloaded: function (video) {
      console.log(video)
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
