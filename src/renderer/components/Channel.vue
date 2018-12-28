<template>
  <div>
    <p><img :src='channel.thumbnail' class="image-round"/>{{ channel.title }}</p>
    <router-link :to="{ name: 'home' }"> back </router-link>
    <el-button @click="getVideos(channel.id)" type="primary">動画リスト取得</el-button>
    <br/>
    <div class="ytb-row" v-for="video in videos">
      <div class="ytb-thumbnail">
        <img :src="video.thumbnail"/>
      </div>
      <div class="ytb-detail">
        <el-button @click="watchVideo(video.id)" type="primary">WATCH</el-button>
        <p>{{ video.title }}</p>
        <p>{{ video.description }}</p>
        <p>{{ video.publishedAt }}</p>
      </div>
    </div>
    <br/>
  </div>
</template>

<script>
import path from 'path'
import { remote } from 'electron'

const {google} = require('googleapis')
const youtube = google.youtube({
  version: 'v3',
  auth: localStorage.getItem('authKey')
})
const {shell} = require('electron')

var Datastore = require('nedb')
var db = new Datastore({
  filename: path.join(remote.app.getPath('userData'), '/video.db'),
  autoload: true
})

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
      var self = this
      db.find({channelId: channelId}, function (err, results) {
        if (err) {
          console.log(err)
        }
        if (results.length > 0) {
          results.sort(function (a, b) {
            if (a.publishedAt > b.publishedAt) return -1
            if (a.publishedAt < b.publishedAt) return 1
            return 0
          })

          self.videos = results
          return
        }
        self.searchVideos(channelId, null, function (videos) {
          self.videos = videos
          db.insert(videos)
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
    searchVideos: function (channelId, nextPageToken, callback) {
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
      var self = this
      youtube.search.list(params, (err, data) => {
        if (err) {
          console.log(err)
        }
        self.videos = data.data.items
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

        self.searchVideos(channelId, next, function (nextList) {
          Array.prototype.push.apply(list, nextList)
          callback(list)
        })
        // callback(list)
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
