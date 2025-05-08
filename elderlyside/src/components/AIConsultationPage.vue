<template>
  <div v-if="isLoggedIn" class="page">
    <!-- Left Sidebar -->
    <el-aside width="220px" class="side">
      <!-- System Name -->
      <div class="sTitle">Smart Home Elderly Care System</div>

      <div class="menuWrap">
        <el-menu
          default-active="/ai-consultation"
          class="el-menu-1"
          :router="true"
        >
          <el-menu-item index="/home">
            <el-icon><Platform /></el-icon>
            <span>Dashboard</span>
          </el-menu-item>
          <el-menu-item index="/health-monitoring">
            <el-icon><Monitor /></el-icon>
            <span>Health monitoring</span>
          </el-menu-item>
          <el-menu-item index="/health-knowledge">
            <el-icon><DocumentCopy /></el-icon>
            <span>Health Knowledge</span>
          </el-menu-item>
          <el-menu-item index="/user-settings">
            <el-icon><Setting /></el-icon>
            <span>User Settings</span>
          </el-menu-item>
          <el-menu-item index="/ai-consultation">
            <el-icon><ChatDotRound /></el-icon>
            <span>AI Consultation</span>
          </el-menu-item>
          <el-menu-item index="/diet-management">
            <el-icon><Food /></el-icon>
            <span>Diet Management</span>
          </el-menu-item>
          <el-menu-item index="/sport-management">
            <el-icon><Basketball /></el-icon>
            <span>Sports Management</span>
          </el-menu-item>
        </el-menu>
      </div>

      <!-- User Info (Bottom) -->
      <div class="uInfo" v-if="userInfo">
        <el-avatar :size="40" :src="userInfo.avatar"></el-avatar>
        <span>{{ userInfo.username }}</span>
      </div>
    </el-aside>

    <!-- Main Content (AI Consultation Page) -->
    <div class="main">
      <!-- Page Title -->
      <div class="pHeader">
        <div class="hTitle">
          <h1>AI Consultation</h1>
          <span class="sub-title">Chat with AI for health advice</span>
        </div>
      </div>

      <el-card class="ai-consultation-card">
        <template #header>
          <div class="card-header">
            <!-- Removed span to make header text bold like previous page -->
            Chat with your health assistant
          </div>
        </template>

        <div class="chat-area">
          <div class="messages-list" ref="messagesList">
            <div
              v-for="(message, index) in conversationHistory"
              :key="index"
              :class="['message-item', message.role]"
            >
              <div class="message-content">
                <p>{{ message.content }}</p>
              </div>
            </div>
            <div
              v-if="consulting"
              class="loading-message message-item assistant"
            >
              <div class="message-content">
                <p>AI is thinking...</p>
              </div>
            </div>
          </div>

          <div class="input-area">
            <el-input
              v-model="userQuestion"
              :rows="2"
              type="textarea"
              placeholder="Enter your message..."
              @keyup.enter="startAiConsultation"
              :disabled="consulting"
            />
            <el-button
              type="primary"
              @click="startAiConsultation"
              :loading="consulting"
              :disabled="!userQuestion.trim() || consulting"
            >
              Send
            </el-button>
          </div>
        </div>

        <div v-if="consultationError" class="error-message">
          {{ consultationError }}
        </div>
      </el-card>
    </div>
  </div>
  <div v-else>
    <h1>Please log in</h1>
    <el-button type="primary" @click="goToLogin">Go to Login</el-button>
  </div>
</template>

<script setup>
import { computed, ref, nextTick, onMounted } from "vue";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import {
  Platform,
  DocumentCopy,
  Setting,
  Monitor,
  ChatDotRound,
  Food,
  Basketball,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus"; // 引入 ElMessage

const router = useRouter();
const store = useStore();

const isLoggedIn = computed(() => store.getters.isLoggedIn);
const userInfo = computed(() => store.getters.userInfo);

const goToLogin = () => {
  router.push("/");
};

const userQuestion = ref("");
const conversationHistory = ref([]);
const consulting = ref(false);
const consultationError = ref(null);
const messagesList = ref(null);

const aiConsultationApiUrl = ref(
  "http://localhost:3060/api/homecare2/ai-consultation"
);

const startAiConsultation = async () => {
  if (!userQuestion.value.trim() || consulting.value) {
    return;
  }

  const question = userQuestion.value;
  userQuestion.value = "";

  conversationHistory.value.push({ role: "user", content: question });
  scrollToBottom();

  consulting.value = true;
  consultationError.value = null;

  try {
    const token = localStorage.getItem("token");
    const response = await fetch(aiConsultationApiUrl.value, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({ question: question }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(
        errorData.message || `HTTP error! status: ${response.status}`
      );
    }

    const data = await response.json();

    if (data.success) {
      conversationHistory.value.push({
        role: "assistant",
        content: data.response,
      });
      scrollToBottom();
    } else {
      consultationError.value = data.message || "AI consultation failed.";
      ElMessage.error(consultationError.value);
      conversationHistory.value.push({
        role: "error",
        content: consultationError.value,
      });
      scrollToBottom();
    }
  } catch (error) {
    console.error("An error occurred during AI consultation:", error);
    consultationError.value =
      error.message ||
      "A network error occurred during AI consultation, please try again later.";
    ElMessage.error(consultationError.value);
    conversationHistory.value.push({
      role: "error",
      content: consultationError.value,
    });
    scrollToBottom();
  } finally {
    consulting.value = false;
  }
};

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesList.value) {
      messagesList.value.scrollTop = messagesList.value.scrollHeight;
    }
  });
};

onMounted(() => {});
</script>

<style scoped>
.page {
  display: flex;
  min-height: 100vh;
  background-color: #f0f2f5;
}

.side {
  width: 220px;
  background-color: #001529;
  border-right: none;
  transition: width 0.28s;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
  position: fixed;
  height: 100vh;
  z-index: 10;
}

.sTitle {
  color: #fff;
  padding: 20px;
  font-size: 18px;
  font-weight: bold;
  text-align: center;
}

.menuWrap {
  height: calc(100vh - 180px);
  overflow: auto;
}

.el-menu {
  border-right: none;
  background-color: #001529;
  flex: 1;
}

.el-menu-item,
.el-sub-menu__title {
  color: rgba(255, 255, 255, 0.65);
}

.el-menu-item:hover,
.el-sub-menu__title:hover {
  background-color: #000c17;
  color: #fff;
}

.el-menu-item.is-active {
  background-color: #1890ff;
  color: #fff;
}

.el-sub-menu.is-active .el-sub-menu__title {
  color: #fff;
}

.el-menu--inline {
  background-color: #000c17 !important;
}

.el-menu--inline .el-menu-item {
  background-color: #000c17 !important;
  color: rgba(255, 255, 255, 0.65);
}

.el-menu--inline .el-menu-item:hover {
  background-color: #001529 !important;
  color: #fff;
}

.el-menu--inline .el-menu-item.is-active {
  background-color: #1890ff !important;
  color: #fff;
}

.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 220px;
  min-height: 100%;
}

.el-menu-item .el-icon,
.el-sub-menu__title .el-icon {
  vertical-align: middle;
  margin-right: 10px;
  width: 24px;
  text-align: center;
  font-size: 18px;
}

.el-sub-menu .el-menu-item {
  min-width: 220px;
  padding-left: 48px !important;
}

.main {
  flex: 1;
  padding: 24px;
  overflow-y: auto;
  margin-left: 220px;
}

.pHeader {
  margin-bottom: 24px;
}

.hTitle h1 {
  margin: 0;
  font-size: 22px;
  color: #303133;
  font-weight: 600;
}

.sub-title {
  color: #606266;
  font-size: 14px;
  margin-top: 4px;
}

.el-card {
  margin-bottom: 24px;
  border-radius: 4px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  color: #303133;
  font-size: 16px;
}

.uInfo {
  display: flex;
  align-items: center;
  padding: 20px;
  color: #fff;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
}

.uInfo span {
  margin-left: 10px;
  font-size: 16px;
}

.ai-consultation-card {
  margin-bottom: 20px; /* Keep consistent margin with other cards */
}

.chat-area {
  display: flex;
  flex-direction: column;
  height: 500px; /* Adjust height as needed */
  border: 1px solid #ebeef5;
  border-radius: 4px;
  overflow: hidden;
}

.messages-list {
  flex: 1;
  overflow-y: auto;
  padding: 15px;
}

.message-item {
  margin-bottom: 10px;
  display: flex;
}

.message-item.user {
  justify-content: flex-end;
}

.message-item.assistant {
  justify-content: flex-start;
}

.message-item.error {
  justify-content: center;
}

.message-content {
  max-width: 70%;
  padding: 10px 15px;
  border-radius: 8px;
  word-break: break-word;
}

.message-item.user .message-content {
  background-color: #409eff;
  color: #fff;
}

.message-item.assistant .message-content {
  background-color: #f4f4f5;
  color: #303133;
}

.message-item.error .message-content {
  background-color: #fef0f0;
  color: #f56c6c;
  border: 1px solid #fde2e2;
}

.input-area {
  display: flex;
  padding: 15px;
  border-top: 1px solid #ebeef5;
  gap: 10px;
}

.input-area .el-button {
  flex-shrink: 0;
}

.loading-message {
  text-align: center;
  color: #409eff;
  margin-bottom: 15px;
}

.error-message {
  color: #f56c6c;
  text-align: center;
  padding: 15px;
  border: 1px solid #f56c6c;
  border-radius: 4px;
  margin-top: 15px;
}
</style>
