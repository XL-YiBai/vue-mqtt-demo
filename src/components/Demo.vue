<script setup lang="ts">
import * as mqtt from "mqtt/dist/mqtt.min";
import { reactive, ref } from "vue";

const isConnecting = ref(false); // 连接状态
let client; // 客户端连接实例
let recevieMessage = ref("");

const connectionInfo = reactive({
  protocol: "ws",
  host: "ibe110e4.cn-hangzhou.emqxcloud.cn",
  clean: true,
  port: 8083,
  username: "xialei",
  password: "public",
  clientId: `mqtt_${Math.random().toString(16).slice(3)}`, // 客户端标识
  connectTimeout: 4000, // 连接超时时长
  reconnectPeriod: 1000, //
});

const subscriptionInfo = reactive({
  topic: "",
  qos: 0,
});

const publishInfo = reactive({
  topic: "",
  qos: 0,
  payload: "",
});

// 创建连接
const handleCreateConnection = () => {
  try {
    let url = `${connectionInfo.protocol}://${connectionInfo.host}:${connectionInfo.port}`;
    url = connectionInfo.protocol.slice(0, 4) == "mqtt" ? url : url + "/mqtt";
    client = mqtt.connect(url, {
      clientId: connectionInfo.clientId,
      clean: true,
      connectTimeout: connectionInfo.connectTimeout,
      username: connectionInfo.username,
      password: connectionInfo.password,
      reconnectPeriod: connectionInfo.reconnectPeriod,
    });
    client.on("connect", () => {});
    listen();
    isConnecting.value = true;
  } catch (error) {
    console.log(error);
  }
};

// 关闭连接
const handleDestoryConnection = () => {};

// 订阅
const handleSubscribe = () => {
  client.subscribe(
    subscriptionInfo.topic,
    { qos: subscriptionInfo.qos },
    (err) => {
      if (err) {
        console.log("subscribe err: " + err);
      } else {
        console.log(`Subscribe to topic '${subscriptionInfo.topic}'`);
      }
    }
  );
};

// 发布
const handlePublish = () => {
  client.publish(
    publishInfo.topic,
    publishInfo.payload,
    {
      qos: publishInfo.qos,
    },
    (err) => {
      if (err) {
        console.log(err);
      }
    }
  );
  console.log(publishInfo.payload);
};

// 监听
function listen() {
  client.on("message", (topic, payload) => {
    recevieMessage.value = recevieMessage.value + payload + "\n";
  });
}
</script>

<template>
  <div class="mqtt-demo">
    <el-card class="card">
      <template #header>
        <div class="card-header">
          <span>建立连接</span>
          <el-button
            v-if="!isConnecting"
            type="primary"
            @click="handleCreateConnection"
            >建立连接</el-button
          >
          <el-button v-else type="danger">关闭连接</el-button>
        </div>
      </template>
      <div class="content">
        <el-form :model="connectionInfo" label-width="120px">
          <el-row :gutter="6">
            <el-col :span="24">
              <el-form-item label="服务器地址：">
                <el-col :span="6">
                  <el-select v-model="connectionInfo.protocol">
                    <el-option label="mqtt://" value="mqtt"></el-option>
                    <el-option label="mqtts://" value="mqtts"></el-option>
                    <el-option label="ws://" value="ws"></el-option>
                    <el-option label="wss://" value="wss"></el-option>
                  </el-select>
                </el-col>
                <el-col :span="18">
                  <el-input v-model="connectionInfo.host"></el-input>
                </el-col>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="端口：">
                <el-input-number v-model="connectionInfo.port">
                </el-input-number>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="Client ID：">
                <el-input v-model="connectionInfo.clientId"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="用户名：">
                <el-input v-model="connectionInfo.username"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="密码：">
                <el-input
                  v-model="connectionInfo.password"
                  type="password"
                ></el-input>
              </el-form-item>
            </el-col>
            <!-- <el-col :span="24">
              <el-form-item label="连接超时时长：">
                <el-input-number
                  v-model="connectionInfo.connectTimeout"
                ></el-input-number
                >&nbsp;ms
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="Keep Alive：">
                <el-input-number
                  v-model="connectionInfo.reconnectPeriod"
                ></el-input-number>
                &nbsp;ms
              </el-form-item>
            </el-col> -->
          </el-row>
        </el-form>
      </div>
    </el-card>
    <el-card class="card">
      <template #header>
        <div class="card-header">
          <span>订阅消息</span>
          <el-button type="primary" @click="handleSubscribe">订阅</el-button>
        </div>
      </template>
      <el-form :model="subscriptionInfo" label-width="120px">
        <el-row :gutter="6">
          <el-col :span="24">
            <el-form-item label="主题(topic)：">
              <el-input v-model="subscriptionInfo.topic"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Qos：">
              <el-input-number
                v-model="subscriptionInfo.qos"
                :min="0"
                :max="2"
                :precision="0"
              ></el-input-number>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
    <el-card class="card">
      <template #header>
        <div class="card-header">
          <span>发布消息</span>
          <el-button type="primary" @click="handlePublish">发布</el-button>
        </div>
      </template>
      <el-form :model="subscriptionInfo" label-width="120px">
        <el-row :gutter="6">
          <el-col :span="24">
            <el-form-item label="主题(topic)：">
              <el-input v-model="publishInfo.topic"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Qos：">
              <el-input-number
                v-model="publishInfo.qos"
                :min="0"
                :max="2"
                :precision="0"
              ></el-input-number>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="Payload：">
              <el-input v-model="publishInfo.payload"> </el-input>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
    <el-card class="card">
      <template #header>
        <span>接收消息</span>
      </template>
      <div>{{ recevieMessage }}</div>
    </el-card>
  </div>
</template>

<style>
.mqtt-demo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card {
  width: 1000px;
  margin-bottom: 40px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
/* 
.el-card__body {
  padding: 0 200px;
} */

/* .content {
  display: flex;
  justify-content: center;
  width: 600px;
} */
</style>
