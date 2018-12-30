<template>
  <div>
    <router-link :to="{ name: 'home' }"> back </router-link>
    <el-form ref="form" :model="form" label-width="120px" label-position="top">
      <el-form-item prop="authKey" label="Youtube AutuKey">
        <el-input placeholder="please input youtube authKey" v-model="form.authKey"></el-input>
      </el-form-item>
      <el-form-item prop="save_dir" label="保存場所">
        <el-input placeholder="please input save dir" v-model="form.saveDir"></el-input>
        <el-button @click="openDialog">dir</el-button>
      </el-form-item>
      <el-button type="primary" @click="submitForm">Save</el-button>
    </el-form>
  </div>
</template>

<script>
const remote = require('electron').remote
const Dialog = remote.dialog

export default {
  name: 'setting',
  data () {
    return {
      form: {
        authKey: '',
        saveDir: ''
      }
    }
  },
  methods: {
    openDialog () {
      Dialog.showOpenDialog(null, {
        properties: ['openDirectory'],
        title: '保存フォルダ選択',
        defaultPath: this.form.saveDir
      }, (folderNames) => {
        this.form.saveDir = folderNames[0]
      })
    },
    submitForm () {
      for (var key in this.form) {
        localStorage.setItem(key, this.form[key])
      }
    }
  },
  mounted () {
    for (var key in this.form) {
      this.form[key] = localStorage.getItem(key)
    }
  }
}
</script>

<style>
.el-input {
  width: 80%;
}

</style>
