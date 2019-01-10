<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="electron-vue">
    <el-table :data="channels" @row-click="rowClicked">
      <el-table-column prop="id" label="id"></el-table-column>
      <el-table-column prop="name" label="name"></el-table-column>
      <el-table-column prop="format" label="format"></el-table-column>
    </el-table>
    <router-link to="/setting">Setting</router-link>

  <!-- 登録ダイアログ -->
  <el-button @click="registry_visible = true">登録</el-button>

  <el-dialog :visible.sync="registry_visible" title="チャンネル登録" width="80%">
    <el-form :model="form" label-width="40px">
      <el-form-item>
        <el-form-item label="ID">
          <el-input placeholder="Please input channel id" v-model="form.id" clearable></el-input>
        </el-form-item>
        <el-form-item label="Name">
          <el-input placeholder="Please input name" v-model="form.name" clearable></el-input>
        </el-form-item>
        <el-form-item label="Type">
          <el-radio-group v-model="form.format">
            <el-radio-button label="mp3"></el-radio-button>
            <el-radio-button label="mp4"></el-radio-button>
          </el-radio-group>
        </el-form-item>
      </el-form-item>
    </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="registry_visible = false">Cancel</el-button>
      <el-button style="float: right" type="primary" @click="addChannel">Add</el-button>
    </span>
  </el-dialog>

  </div>
</template>

<script>
import db from '../datastore.js'
import dbData from './db/channellist.json'

export default {
  name: 'home',
  data () {
    return {
      form: {
        channelId: '',
        name: '',
        format: 'mp3'
      },
      registry_visible: false,
      channels: []
    }
  },
  methods: {
    addChannel () {
      if (this.form.id === '' || this.form.name === '') {
        return
      }
      var data = {'id': this.form.id, 'name': this.form.name, 'format': this.form.format}
      this.channels.push(data)
      this.form = {}
      this.$db.insert(data)
      this.registry_visible = false
    },
    rowClicked (row) {
      this.$router.push({name: 'channel', params: { id: row.id }})
    }
  },
  mounted () {
    db.find({}, function (err, channels) {
      if (err) {
        console.log(err)
      }
      if (channels.length === 0) {
        db.insert(dbData)
        channels = dbData
      }
      this.channels = channels
    }.bind(this))
  }
}
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }

  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 60px 80px;
    width: 100vw;
  }

  #logo {
    height: auto;
    margin-bottom: 20px;
    width: 420px;
  }

  main {
    display: flex;
    justify-content: space-between;
  }

  main > div { flex-basis: 50%; }

  .left-side {
    display: flex;
    flex-direction: column;
  }

  .welcome {
    color: #555;
    font-size: 23px;
    margin-bottom: 10px;
  }

  .title {
    color: #2c3e50;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 6px;
  }

  .title.alt {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .doc p {
    color: black;
    margin-bottom: 10px;
  }

  .doc button {
    font-size: .8em;
    cursor: pointer;
    outline: none;
    padding: 0.75em 2em;
    border-radius: 2em;
    display: inline-block;
    color: #fff;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    box-sizing: border-box;
    border: 1px solid #4fc08d;
  }

  .doc button.alt {
    color: #42b983;
    background-color: transparent;
  }
</style>
