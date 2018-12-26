<template>
  <div id="wrapper">
    <img id="logo" src="~@/assets/logo.png" alt="electron-vue">
    <el-form :model="form" label-width="40px">
      <el-form-item>
        <el-form-item label="ID">
          <el-input placeholder="Please input channel id" v-model="form.id" clearable></el-input>
        </el-form-item>
        <el-form-item label="Name">
          <el-input placeholder="Please input name" v-model="form.name" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button style="float: right" type="primary" @click="addChannel">Add</el-button>
        </el-form-item>
      </el-form-item>
    </el-form>
    <el-table :data="channels" @row-click="rowClicked">
      <el-table-column prop="id" label="id"></el-table-column>
      <el-table-column prop="name" label="name"></el-table-column>
    </el-table>
    <br/>
    <el-form :model="setting" label-width="40px">
      <el-form-item>
        <el-form-item label="auth">
          <el-input placeholder="Please input auth key" v-model="setting.authKey" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button style="float: right" type="primary" @click="updateSetting">Settiing</el-button>
        </el-form-item>
      </el-form-item>
    </el-form>
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
        name: ''
      },
      channels: [],
      setting: {
        authkey: ''
      }
    }
  },
  methods: {
    addChannel () {
      if (this.form.id === '' || this.form.name === '') {
        return
      }
      var data = {'id': this.form.id, 'name': this.form.name}
      this.channels.push(data)
      this.form = {}
      this.$db.insert(data)
    },
    rowClicked (row) {
      this.$router.push({name: 'channel', params: { id: row.id }})
    },
    updateSetting () {
      localStorage.setItem('authKey', this.setting.authKey)
      console.log('update setting')
      console.log(localStorage.getItem('authKey'))
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

    this.setting.authKey = localStorage.getItem('authKey')
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
