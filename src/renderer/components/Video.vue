<template>
  <div class="ytb-row">
    <div class="ytb-thumbnail">
      <img :src="video.thumbnail"/>
    </div>
    <div class="ytb-detail">
      <el-button @click="watchVideo(video.id)" type="primary">WATCH</el-button>
      <el-button @click="downloadVideo(video.id, video.title)" type="warning">DL</el-button>
      <p>{{ video.title }}</p>
      <p>{{ video.description }}</p>
      <p>{{ video.publishedAt }}</p>
    </div>
  </div>
</template>

<script>
const {shell} = require('electron')
export default {
  props: {
    video: {type: Object, default: () => { return {} }}
  },
  methods: {
    watchVideo: function (videoId) {
      shell.openExternal('https://www.youtube.com/watch?v=' + videoId)
    },
    downloadVideo: function (videoId, title) {
      const ytdl = require('ytdl-core')
      const BASE_PATH = `https://www.youtube.com/watch?v=`
      const url = BASE_PATH + videoId
      const ffmpeg = require('fluent-ffmpeg')

      var saveDir = localStorage.getItem('saveDir')
      var savePath = saveDir + '/' + title.replace(/\//g, '_') + '.mp3'

      var reader = ytdl(url, {filter: 'audioonly'})
      var writer = ffmpeg(reader).format('mp3').audioBitrate(128)

      writer
        .on('end', () => {
          this.$emit('video-downloaded', this.video)
        })

      writer.output(savePath).run()
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
