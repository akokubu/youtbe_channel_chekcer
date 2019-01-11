<template>
  <div class="ytb-row">
    <div class="ytb-thumbnail">
      <img :src="video.thumbnail"/>
    </div>
    <div class="ytb-detail">
      <el-button @click="watchVideo(video.id)" type="primary">WATCH</el-button>
      <el-button v-if="downloading" type="primary">Downloadind...</el-button>
      <el-button v-else @click="downloadVideo(video.id, video.title)" :type="downloadedVideo">DL</el-button>
      <p>{{ video.title }}</p>
      <p>{{ video.description }}</p>
      <p>{{ video.publishedAt }}</p>
      <el-checkbox @change="downloadedClick" v-model="video.downloaded">downloaded</el-checkbox>
    </div>
  </div>
</template>

<script>
const {shell} = require('electron')
export default {
  props: {
    video: {type: Object, default: () => { return {} }},
    format: String
  },
  data () {
    return {
      downloading: false
    }
  },
  methods: {
    watchVideo: function (videoId) {
      shell.openExternal('https://www.youtube.com/watch?v=' + videoId)
    },
    downloadVideo: function (videoId, title) {
      this.downloading = true

      const fs = require('fs')
      const ytdl = require('ytdl-core')
      const BASE_PATH = `https://www.youtube.com/watch?v=`
      const url = BASE_PATH + videoId
      const ffmpeg = require('fluent-ffmpeg')

      var saveDir = localStorage.getItem('saveDir')
      var mp4SavePath

      if (this.format === 'mp3') {
        mp4SavePath = '/tmp/' + videoId + '.mp4'
      } else {
        mp4SavePath = saveDir + '/' + title.replace(/\//g, '_') + '.mp4'
      }

      const mp4Video = ytdl(url)
      mp4Video.pipe(fs.createWriteStream(mp4SavePath))
      mp4Video.on('end', () => {
        // convert mp3
        if (this.format === 'mp3') {
          const proc = ffmpeg({source: mp4SavePath})
          var savePath = saveDir + '/' + title.replace(/\//g, '_') + '.mp3'
          proc.format('mp3').audioBitrate(128)
          proc.on('end', () => {
            this.downloading = false
            this.$set(this.video, 'downloaded', true)
            this.$emit('video-downloaded', this.video)
          })
          proc.output(savePath).run()
        } else {
          this.downloading = false
          this.$set(this.video, 'downloaded', true)
          this.$emit('video-downloaded', this.video)
        }
      })
    },
    downloadedClick: function () {
      this.$emit('video-downloaded', this.video)
    }
  },
  computed: {
    downloadedVideo: function () {
      return this.video.downloaded ? 'warning' : 'danger'
    }
  }
}
</script>

<style>
.ytb-thumbnail {
  display: inline-block;
  position: relative;
  padding-right: 20px;
}
.ytb-row {
  display: flex;
  margin-bottom: 10px;
}
</style>
