<template>
  <el-container class="layout-container">
    <!-- Sidebar menu - Kept unchanged -->
    <el-aside width="240px" class="sidebar">
      <div class="logo">
        <span class="logo-text">VCIS大模型生成内容检测平台</span>
      </div>
      <el-menu
        :default-active="activeMenu"
        background-color="#1e1e2d"
        text-color="#a2a3b7"
        active-text-color="#ffffff"
        @select="handleSelect"
      >
        <el-menu-item index="llm">
          <i class="el-icon-setting"></i>
          <el-icon><MessageBox /></el-icon>
          <span>大模型配置</span>
        </el-menu-item>
        <el-menu-item index="dateset">
          <i class="el-icon-document"></i>
          <el-icon><Edit /></el-icon>
          <span>数据集选择</span>
        </el-menu-item>
        <el-menu-item index="task">
          <i class="el-icon-plus"></i>
          <el-icon><Notification /></el-icon>
          <span>发布任务</span>
        </el-menu-item>
        <el-menu-item index="taskmanger">
          <i class="el-icon-s-order"></i>
          <el-icon><Compass /></el-icon>
          <span>任务管理</span>
        </el-menu-item>
      </el-menu>
    </el-aside>

    <!-- Main content area - Optimized -->
    <el-container>
      <el-header class="header">
        <h2 class="page-title">{{ currentPageTitle }}</h2>
        <div class="user-info">
          <el-avatar src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"></el-avatar>
          <span class="username">用户名</span>
        </div>
      </el-header>
      <el-main class="main-content">
        <div class="steps-container">
          <el-steps :active="0" finish-status="success" align-center>
            <el-step title="大模型配置" />
            <el-step title="数据集选择" />
            <el-step title="发布任务" />
            <el-step title="任务管理" />
          </el-steps>
        </div>
        
        <el-card class="config-card">
          <el-form label-position="top" :model="formData" class="config-form">
            <el-form-item label="公司、团队或机构名称">
              <el-input v-model="formData.teamName" placeholder="请输入公司、组织或机构名称"></el-input>
            </el-form-item>
            <el-form-item label="大模型名称">
              <el-input v-model="formData.modelName" placeholder="大模型名称"></el-input>
            </el-form-item>
            <el-form-item label="大模型API调用方式">
              <el-input v-model="formData.apiConfig" type="textarea" :rows="15" placeholder="请输入API配置（JSON格式）"></el-input>
            </el-form-item>
          </el-form>
          <div class="form-actions">
            <el-button type="primary" @click="saveModelConfig">保存</el-button>
            <el-button type="success" @click="goToNextStep">下一步</el-button>
          </div>
        </el-card>
      </el-main>
    </el-container>
  </el-container>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

export default defineComponent({
  name: 'ModelConfiguration',
  setup() {
    const router = useRouter();
    const activeMenu = ref('llm');
    const currentPageTitle = computed(() => '大模型配置');

    const formData = ref({
      teamName: '',
      modelName: '',
      apiConfig: JSON.stringify({
        modelConfig: {
          url: "http://xxx.yyy.zzz/api",
          headers: {
            "Content-Type": "application/json",
            "Access-Key": "xxx"
          },
          data: {
            prompt: "%prompt%",
            token: "xxx"
          },
          answer: "result.content",
          concurrency: 2,
          retries: 1,
          timeout: 60
        },
        userJwt: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE3NTI4OTE4NzF9.6OFjQ62TRDJbW4fdGvuhm3lKazws_iUUGrVKInPDMt8"
      }, null, 2)
    });

    const saveModelConfig = async () => {
      try {
        const config = JSON.parse(formData.value.apiConfig);
        const requestData = {
          modelName: formData.value.modelName,
          modelConfig: config.modelConfig,
          userJwt: config.userJwt
        };
        console.log("Request Data:", requestData);
        const response = await axios.post('http://10.110.147.246:5004/model/add', requestData);
        console.log('Response:', response.data);
        // 可以在这里添加成功提示
      } catch (error) {
        console.error('Error:', error);
        // 可以在这里添加错误提示
      }
    };

    const goToNextStep = () => {
      router.push('/dateset');
    };

    const handleSelect = (key: string) => {
      router.push(`/${key}`);
    };

    return {
      formData,
      activeMenu,
      currentPageTitle,
      saveModelConfig,
      goToNextStep,
      handleSelect
    };
  }
});
</script>

<style scoped>
.layout-container {
  height: 100vh;
  display: flex;
  font-family: 'Roboto', sans-serif;
}

.sidebar {
  background-color: #1e1e2d;
  display: flex;
  flex-direction: column;
}

.logo {
  padding: 20px;
  text-align: center;
  background-color: #1e1e2d;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.logo-text {
  display: block;
  color: #ffffff;
  margin-top: 20px;
  font-family: 'Arial', sans-serif; /* 使用更现代的字体 */
  font-size: 24px; /* 增大字体大小 */
  font-weight: bold; /* 加粗文本 */
  letter-spacing: 1px; /* 增加字母间距 */
  text-transform: uppercase; /* 将文本转换为大写 */
}


.header {
  background-color: #ffffff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 30px;
  height: 70px;
}
.sidebar .el-menu-item span {
  font-size: 16px;
  font-weight: 500;
}
.page-title {
  font-size: 24px;
  color: #1e1e2d;
  font-weight: 500;
}

.user-info {
  display: flex;
  align-items: center;
}

.username {
  margin-left: 15px;
  font-size: 16px;
  color: #101111;
  font-weight: 500;
}

.main-content {
  padding: 30px;
  background-color: #f5f7fa;
}

.steps-container {
  margin-bottom: 30px;
}

.config-card {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.config-form {
  max-width: 800px;
  margin: 0 auto;
}

.form-actions {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.form-actions .el-button {
  margin: 0 10px;
}
</style>