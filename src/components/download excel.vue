<script setup>
import { ref } from 'vue';
import * as XLSX from 'xlsx';

// 定义props
const props = defineProps({
    autoClosePrompt: {
    type: Boolean,
    default: true
    }
});

// 状态管理
const isDownloading = ref(false);
const downloadError = ref('');
const downloadSuccess = ref(false);
const showSuccessModal = ref(false); // 控制成功弹出窗口的显示
const showLoadingModal = ref(false); // 控制下载中弹出窗口的显示
const showErrorModal = ref(false); // 控制错误弹出窗口的显示

// 事件
const emit = defineEmits(['download-complete', 'download-error']);

// 下载Google试算表
const downloadGoogleSheet = async (spreadsheetId) => {
    if (!spreadsheetId) {
    downloadError.value = '請提供正確的班表ID';
    emit('download-error', downloadError.value);
    return null;
    }

    isDownloading.value = true;
    downloadError.value = '';
    downloadSuccess.value = false;
    showLoadingModal.value = true; // 显示下载中弹出窗口

    try {
    // 构建Google试算表导出URL
    const exportUrl = `https://docs.google.com/spreadsheets/d/${spreadsheetId}/export?format=xlsx`;

    // 下载Excel文件
    const response = await fetch(exportUrl);

    if (!response.ok) {
        throw new Error(`下載失敗: ${response.status} ${response.statusText}`);
    }

    // 获取二进制数据
    const arrayBuffer = await response.arrayBuffer();
    const data = new Uint8Array(arrayBuffer);

    // 使用XLSX解析Excel数据
    const workbook = XLSX.read(data, { type: 'array' });
    const sheetName = workbook.SheetNames[0];
    const sheetData = XLSX.utils.sheet_to_json(workbook.Sheets[sheetName], { header: 1 });

    // 下载成功
    downloadSuccess.value = true;
    // 显示成功弹出窗口
    showSuccessModal.value = true;
    // 只有当autoClosePrompt为true时才自动关闭弹出窗口
    if (props.autoClosePrompt) {
        setTimeout(() => {
        showSuccessModal.value = false;
        }, 3000);
    }

    // 触发下载完成事件，传递解析后的数据
    emit('download-complete', {
        workbook,
        sheetData,
        fileName: `志工班表_${new Date().toISOString().split('T')[0]}.xlsx`
    });

    return {
      workbook,
      sheetData
    };
  } catch (error) {
    console.error('下載班表失敗:', error);
    downloadError.value = `下載失敗: ${error.message}`;
    emit('download-error', downloadError.value);
    showErrorModal.value = true; // 显示错误弹出窗口
    // 只有当autoClosePrompt为true时才自动关闭弹出窗口
    if (props.autoClosePrompt) {
      setTimeout(() => {
        showErrorModal.value = false;
      }, 5000);
    }
    return null;
  } finally {
    isDownloading.value = false;
    showLoadingModal.value = false; // 隐藏下载中弹出窗口
  }
};

// 导出函数供其他组件使用
defineExpose({
  downloadGoogleSheet
});
</script>

<template>
  <div class="download-container" v-if="isDownloading || downloadError || downloadSuccess">
    <!-- 下载状态信息容器 -->
  </div>
  
  <!-- 下载中弹出窗口 -->
  <div class="modal-overlay" v-if="showLoadingModal">
    <div class="modal-container loading-modal" @click.stop>
      <div class="modal-header">
        <h2 class="modal-title"><span class="loading-icon">⏳</span> 下載中</h2>
      </div>
      <div class="modal-body">
        <div class="loading-message">
          <div class="loading-spinner"></div>
          <span class="loading-text">正在下載班表中...</span>
          <div class="loading-details">請稍後，班表載入中...</div>
        </div>
      </div>
    </div>
  </div>
  
  <!-- 错误弹出窗口 -->
  <div class="modal-overlay" v-if="showErrorModal" @click="showErrorModal = false">
    <div class="modal-container error-modal" @click.stop>
      <div class="modal-header">
        <h2 class="modal-title"><span class="error-icon">❌</span> 下載失敗</h2>
        <button class="close-button" @click="showErrorModal = false">×</button>
      </div>
      <div class="modal-body">
        <div class="error-message">
          <span class="error-text">班表下載失敗</span>
          <div class="error-details">{{ downloadError }}</div>
        </div>
      </div>
      <div class="modal-footer">
        <button class="modal-button error" @click="showErrorModal = false">確定</button>
      </div>
    </div>
  </div>
  
  <!-- 成功弹出窗口 -->
  <div class="modal-overlay" v-if="showSuccessModal" @click="showSuccessModal = false">
    <div class="modal-container success-modal" @click.stop>
      <div class="modal-header">
        <h2 class="modal-title"><span class="success-icon">✅</span> 下載成功</h2>
        <button class="close-button" @click="showSuccessModal = false">×</button>
      </div>
      <div class="modal-body">
        <div class="success-message">
          <span class="success-text">班表下載成功</span>
          <div class="success-details">內容已載入，可繼續操作。</div>
        </div>
      </div>
      <div class="modal-footer">
        <button class="modal-button save" @click="showSuccessModal = false">確定</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.download-container {
  /*margin: 10px 0;
  padding: 10px;*/
  border-radius: 8px;
  background-color: #f8f9fa;
}

.download-status {
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 14px;
  display: flex;
  align-items: center;
  margin-bottom: 8px;
}

.status-icon {
  margin-right: 8px;
  font-size: 16px;
}

.downloading {
  background-color: #e3f2fd;
  color: #0d47a1;
}

.error {
  background-color: #ffebee;
  color: #c62828;
}

.success {
  background-color: #e8f5e9;
  color: #2e7d32;
}

/* 模态窗口样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-container {
  background-color: white;
  border-radius: 16px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  animation: slideIn 0.3s ease;
}

.success-modal {
  border-top: 5px solid #4caf50;
}

.loading-modal {
  border-top: 5px solid #2196f3;
}

.error-modal {
  border-top: 5px solid #f44336;
}

@keyframes slideIn {
  from { transform: translateY(-50px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.modal-header {
  padding: 15px 20px;
  border-bottom: 1px solid #e2e8f0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-title {
  margin: 0;
  font-size: 20px;
  color: #2d3748;
  display: flex;
  align-items: center;
}

.success-icon {
    margin-right: 10px;
    font-size: 24px;
    color: #4caf50;
}

.loading-icon {
    margin-right: 10px;
    font-size: 24px;
    color: #2196f3;
}

.error-icon {
    margin-right: 10px;
    font-size: 24px;
    color: #f44336;
}

.close-button {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #718096;
    transition: color 0.2s;
}

.close-button:hover {
  color: #e53e3e;
}

.modal-body {
  padding: 20px;
}

.success-message {
  text-align: center;
  padding: 10px 0;
}

.success-text {
  font-size: 18px;
  font-weight: 600;
  color: #2e7d32;
  display: block;
  margin-bottom: 10px;
}

.loading-text {
  font-size: 18px;
  font-weight: 600;
  color: #0d47a1;
  display: block;
  margin-bottom: 10px;
}

.error-text {
  font-size: 18px;
  font-weight: 600;
  color: #c62828;
  display: block;
  margin-bottom: 10px;
}

.success-details {
  color: #718096;
  font-size: 14px;
}

.loading-details {
  color: #718096;
  font-size: 14px;
}

.error-details {
  color: #c62828;
  font-size: 14px;
  word-break: break-word;
}

.modal-footer {
  padding: 15px 20px;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.modal-button {
  padding: 10px 20px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.modal-button.save {
  background: linear-gradient(to right, #4facfe, #00f2fe);
  color: white;
  border: none;
  box-shadow: 0 4px 6px rgba(66, 153, 225, 0.3);
}

.modal-button.error {
  background: linear-gradient(to right, #ff416c, #ff4b2b);
  color: white;
  border: none;
  box-shadow: 0 4px 6px rgba(229, 62, 62, 0.3);
}

.modal-button.save:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(66, 153, 225, 0.4);
}

.modal-button.error:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(229, 62, 62, 0.4);
}

/* 加载动画 */
.loading-spinner {
  display: inline-block;
  width: 50px;
  height: 50px;
  border: 4px solid rgba(33, 150, 243, 0.2);
  border-radius: 50%;
  border-top-color: #2196f3;
  animation: spin 1s ease-in-out infinite;
  margin-bottom: 15px;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-message {
  text-align: center;
  padding: 10px 0;
}
</style>