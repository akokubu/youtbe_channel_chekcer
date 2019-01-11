<template>
  <div>
    <p><img :src='channel.thumbnail' class="image-round"/>{{ channel.title }}</p>
    <router-link :to="{ name: 'home' }"> back </router-link>
    <el-radio-group v-model="channel.format" @change="changeFormat">
      <el-radio-button label="mp3"></el-radio-button>
      <el-radio-button label="mp4"></el-radio-button>
    </el-radio-group>
    <hr/>
    <el-button @click="getUnDownloadedVideos(channel.id)" type="primary">未ダウンロード動画リスト取得</el-button>
    <el-button @click="getVideos(channel.id)" type="primary">動画リスト取得</el-button>
    <el-button @click="deleteVideo(channel.id)" type="danger">リスト削除</el-button>
    <br/>
    <div v-for="video in videos" :key="video._id">
      <yt-video :video="video" :format="channel.format" v-on:video-downloaded="videoDownloaded"></yt-video>
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
var videoDb = new Datastore({
  filename: path.join(remote.app.getPath('userData'), '/video.db'),
  autoload: true
})

export default {
  components: {
    YtVideo
  },
  data () {
    return {
      channel: {},
      videos: []
    }
  },
  methods: {
    deleteVideo: function (channelId) {
      console.log(channelId)
      videoDb.remove({channelId: channelId}, {multi: true}, function (err, numRemoved) {
        if (err) {
          console.log(err)
          return
        }
        console.log('deleted: ' + numRemoved)
      })
    },
    getUnDownloadedVideos: function (channelId) {
      this.getVideos(channelId, true)
    },
    getVideos: function (channelId, undownloaded = false) {
      var self = this
      videoDb.find({channelId: channelId}, function (err, results) {
        if (err) {
          console.log(err)
          return
        }
        var lastUpdate = null
        if (results.length > 0) {
          results.sort(function (a, b) {
            if (a.publishedAt > b.publishedAt) return -1
            if (a.publishedAt < b.publishedAt) return 1
            return 0
          })

          var dt = new Date(results[0].publishedAt)
          dt.setSeconds(dt.getSeconds() + 1)
          lastUpdate = dt.toISOString().split('.')[0] + 'Z'

          if (undownloaded) {
            results = results.filter(video => video.downloaded !== true)
          }
          self.videos = results
        }
        self.searchVideos(channelId, lastUpdate, null, function (videos) {
          if (videos.length > 0) {
            videoDb.insert(videos, function (err, newvideos) {
              if (err) {
                console.log(err)
                return
              }
              Array.prototype.push.apply(newvideos, results)
              self.videos = newvideos
            })
          }
        })
      })
    },
    getChannel (channelId) {
      var self = this
      this.$db.findOne({id: channelId}, function (err, channel) {
        if (err) {
          console.log(err)
          return
        }

        const params = {
          part: 'snippet',
          id: channelId
        }

        youtube.channels.list(params, (err, data) => {
          if (err) {
            console.log(err)
            return
          }
          var ch = data.data.items[0].snippet
          self.channel = {
            id: params.id,
            title: ch.title,
            name: channel.name,
            thumbnail: ch.thumbnails.medium.url,
            format: channel.format
          }
        })
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
      videoDb.update({ _id: video._id }, { $set: { downloaded: video.downloaded } }, {}, function () {})
    },
    changeFormat: function () {
      this.$db.update({ id: this.channel.id }, { $set: { format: this.channel.format } }, {}, function () {})
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
