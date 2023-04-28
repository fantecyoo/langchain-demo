<template>
  <div>
    <el-tabs v-model="activeName" class="demo-tabs" @tab-click="handleClick">
      <el-tab-pane label="Search" name="first">
        <HelloWorld type="search" />
      </el-tab-pane>
      <el-tab-pane label="Chat" name="second">
        <HelloWorld type="chat" />
      </el-tab-pane>
      <!-- <el-tab-pane label="Text" name="thrid">
        <el-input
          v-model="textarea"
          :rows="25"
          type="textarea"
          placeholder="Please input"
        />
        <br />
        <br />
        <el-button @click="uploadText" type="primary">Upload</el-button>
      </el-tab-pane> -->
    </el-tabs>
  </div>
</template>

<script setup>
import { ElTabs, ElInput, ElButton } from "element-plus"
import { ref } from "vue"
import HelloWorld from "./HelloWorld.vue"
const activeName = ref("first")

function handleClick(tab, event) {
  console.log(tab, event)
}

function uploadText() {
  console.log(textarea)
}
</script>

<style lang="scss" scoped></style>
