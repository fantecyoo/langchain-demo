<template>
  <div class="container">
    <div class="header"></div>
    <div class="content" v-if="props.type === 'chat'">
      <div class="conversation">
        <div v-for="msg in messageChain" :key="msg.id" class="msg">
          <div :class="getMsgClass(msg)">{{ msg.content }}</div>
        </div>
      </div>
    </div>
    <div v-else-if="props.type === 'search'">
      <el-table :data="searchResult">
        <el-table-column>
          <!-- <template #header>
            <span>相似度</span>
          </template> -->
          <template #default="{ row }">
            <span style="font-weight: bold">{{ row.metadata.source }}</span>
            <div>{{ row.pageContent }}</div>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="footer">
      <div class="loadingInfo" v-loading="loading"></div>
      <div class="input">
        <el-input
          v-model="keyword"
          placeholder="请输入关键词"
          @keyup.enter="search"
        ></el-input>
        <el-button @click="search">搜索</el-button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue"
import { ElButton, ElInput } from "element-plus"
const messageChain = ref([])
const keyword = ref("")
const loading = ref(false)
const time = ref(0)
// 获取props
const props = defineProps({
  msg: String,
  type: String
})

const searchResult = ref([])

const getMsgClass = msg => {
  let className = msg.type === "robot" ? "robotMsg" : "userMsg"
  return {
    [className]: true
  }
}

const search = async () => {
  // 以时间戳为id生成一条message，加到messageChain中
  let id = new Date().getTime()
  messageChain.value.push({
    id,
    content: keyword.value,
    type: "user"
  })
  let searchKeyword = keyword.value
  keyword.value = ""
  loading.value = true
  // 记录时间戳
  time.value = id

  // await fetch("api/data")
  //   .then(response => {
  //     // 这里拿到的 response 并不是一个 {name: 'test', age: 1} 对象
  //     return response.json() // 将 response.body 通过 JSON.parse 转换为 JS 对象
  //   })
  //   .then(data => {
  //     console.log(data) // {name: 'test', age: 1}
  //   })
  // console.log(searchKeyword)

  // similarity-search
  if (props.type === "search") {
    await getSimilarInfo(searchKeyword)
  } else if (props.type === "chat") {
    await getAnswer(searchKeyword)
  }
}

async function getSimilarInfo(searchKeyword) {
  const response = await fetch("api/similarity-search", {
    method: "POST",
    headers: {
      "Content-type": "application/json;charset=utf-8"
    },
    body: JSON.stringify({
      content: searchKeyword
    })
  })
  const json = await response.json()
  console.log(json)
  if (json.length > 0) {
    searchResult.value = json
  }
  loading.value = false
}

async function getAnswer(searchKeyword) {
  const response = await fetch("api/gpt", {
    method: "POST",
    headers: {
      "Content-type": "application/json;charset=utf-8"
    },
    body: JSON.stringify({
      content: searchKeyword
    })
  })
  const json = await response.json()
  console.log(json)

  loading.value = false
  if (json.text) {
    messageChain.value.push({
      id: new Date().getTime(),
      content: json.text,
      type: "robot"
    })
  }
}
</script>

<style lang="scss" scoped>
.container {
  padding: 60px;
  display: flex;
  flex-direction: column;
}
.input {
  display: flex;
  gap: 20px;
  .el-input {
    flex: 1;
  }
}
.conversation {
  .msg {
    display: flex;
    flex-direction: column;
    margin-bottom: 20px;
  }
  .robotMsg,
  .userMsg {
    border-radius: 12px;
    // max-width: 768px;
    padding: 10px 16px;
    display: inline-block;
  }
  .robotMsg {
    background: linear-gradient(90deg, #dcdbdc 10.79%, #cfcfcf 87.08%);
    align-self: flex-start;
  }
  .userMsg {
    background: linear-gradient(90deg, #2870ea 10.79%, #1b4aef 87.08%);
    color: #fff;
    align-self: flex-end;
  }
}
.content {
  height: calc(100vh - 320px);
  border-radius: 12px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  overflow: auto;
  padding: 20px;
  position: relative;
  .fade {
    height: 30px;
    left: 60px;
    right: 60px;
    -webkit-mask-image: linear-gradient(black 0px, transparent 30px);
    position: fixed;
    top: 150px;
    z-index: 1;
    // background-color: black;
  }
}

.footer {
  .loadingInfo {
    text-align: center;
    height: 50px;
  }
  position: fixed;
  width: calc(100% - 80px);
  bottom: 30px;
  z-index: 1;
}
</style>
