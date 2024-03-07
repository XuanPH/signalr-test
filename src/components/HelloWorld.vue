<script setup>
import {reactive, ref} from 'vue'
import {UserOutlined, EyeTwoTone, EyeInvisibleOutlined, LinkOutlined} from "@ant-design/icons-vue";
import {HttpTransportType, HubConnectionBuilder, LogLevel} from '@microsoft/signalr';
import {HubConnectionState} from "@microsoft/signalr/dist/esm/HubConnection.js";

defineProps({
  msg: String,
})

const count = ref(0)
const dataSignal = reactive({
  url: 'http://115.165.166.123:2222/portal-hub/chat',
  vsss_id: 'P2300253',
  api_key: '2mHi6A/x8jydHVpaOFPwMJ72ft382vr0hFkfS4pS/ets=.39155d2f-db96-11ee-b60f-0242ac140002'
})
const logs = ref([]);
console.stdlog = console.log.bind(console);
console.log = function(){
  if (arguments[0].includes("Debug:")) {
    logs.value.push(Array.from(arguments));
  }
  console.stdlog.apply(console, arguments);
}

const connection = ref();
const onClear = () => {
  logs.value = [];
  if (connection.value && connection.value.state == HubConnectionState.Connected) {
    connection.value.stop();
  }
}
const onConnect = () => {
  if (connection.value && connection.value.state === HubConnectionState.Connected) {
    connection.value.stop();
  }
  let urlBuilder = `${dataSignal.url}?vsss_id=${dataSignal.vsss_id}&api_key=${dataSignal.api_key}`
  connection.value = new HubConnectionBuilder()
      .withUrl(urlBuilder, {
        skipNegotiation: true,
        transport: HttpTransportType.WebSockets,
      })
      .configureLogging(LogLevel.Debug)
      .build();
  connection.value.start();
  connection.value.on("ping", (data) => {
    console.log("Connection ping:" + data);
  });
  setTimeout(() => {
    console.log(connection.value._logger.out)
  }, 3000)
}

</script>

<template>
  <h1>{{ msg }}</h1>
  <div style="width: 700px">
    <div class="components-input-demo-presuffix">
      <a-input v-model:value="dataSignal.url" placeholder="Url">
        <template #suffix>
          <LinkOutlined/>
        </template>
      </a-input>
      <br/>
      <br/>
      <a-input v-model:value="dataSignal.vsss_id" placeholder="VSSS Id">
        <template #suffix>
          <UserOutlined/>
        </template>
      </a-input>
      <br/>
      <br/>
      <a-input-password v-model:value="dataSignal.api_key" placeholder="API Key">
        <template #iconRender="v">
          <EyeTwoTone v-if="v"></EyeTwoTone>
          <EyeInvisibleOutlined v-else></EyeInvisibleOutlined>
        </template>
      </a-input-password>
    </div>
    <div class="card" style="text-align: left; max-height: 370px; overflow: auto">
      <div v-for="(item,index) in logs" :style="{ color: item[0].includes('error') ? 'red' :
      (item[0].includes('successfully') ? 'green' : 'blue')
      }" :key="index" style="border-bottom: 1px dotted black">
        {{ item[0] }}
      </div>
    </div>
  </div>
  <div class="card">
    <button type="button" @click="onConnect">Connect to the server</button>
    <button style="margin-left: 50px" type="button" @click="onClear">Clear Connection</button>
  </div>

</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
