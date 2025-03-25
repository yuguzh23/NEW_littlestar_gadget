<script setup>
import { ref, defineProps, defineEmits } from 'vue';

// 定義props和emits
const props = defineProps({
    googleSheetId: {
        type: String,
        default: ''
    },
    excelDateRange: {
        type: String,
        default: 'C1~N1'
    },
    volunteerRowRange: {
        type: String,
        default: '6~55'
    },
    autoClosePrompt: {
        type: Boolean,
        default: true
    }
});

const emit = defineEmits(['update:googleSheetId', 'update:excelDateRange', 'update:volunteerRowRange', 'update:autoClosePrompt', 'save-settings', 'close']);

// 本地狀態，用於雙向綁定
const localGoogleSheetId = ref(props.googleSheetId);
const localExcelDateRange = ref(props.excelDateRange);
const localVolunteerRowRange = ref(props.volunteerRowRange);
const localAutoClosePrompt = ref(props.autoClosePrompt);

// 監聽props變化
const updateLocalValues = () => {
    localGoogleSheetId.value = props.googleSheetId;
    localExcelDateRange.value = props.excelDateRange;
    localVolunteerRowRange.value = props.volunteerRowRange;
    localAutoClosePrompt.value = props.autoClosePrompt;
};

// 保存設定
const saveSettings = () => {
    // 發出更新事件
    emit('update:googleSheetId', localGoogleSheetId.value);
    emit('update:excelDateRange', localExcelDateRange.value);
    emit('update:volunteerRowRange', localVolunteerRowRange.value);
    emit('update:autoClosePrompt', localAutoClosePrompt.value);
    
    // 發出保存事件
    emit('save-settings');
    
    // 不需要關閉模態視窗，由父組件處理
};
</script>

<template>
    <!-- 設定模態視窗 -->
    <div class="modal-overlay">
        <div class="modal-container" @click.stop>
            <div class="modal-header">
                <h2 class="modal-title"><span class="settings-icon">⚙️</span> 系統設定</h2>
                <button class="close-button" @click="$emit('close')">×</button>
            </div>
            <div class="modal-body">
                <div class="form-group">
                    <label for="modalGoogleSheetId"><span class="label-icon">📊</span> 班表ID：</label>
                    <input 
                        type="text" 
                        id="modalGoogleSheetId" 
                        v-model="localGoogleSheetId" 
                        placeholder="請輸入班表ID" 
                        class="modal-input"
                    />
                    <div class="input-help">
                        <span class="info-icon">ℹ️</span> 請從班表網址中複製ID，例如：https://docs.google.com/spreadsheets/d/<strong>1AbCdEfGhIjKlMnOpQrStUvWxYz</strong>/edit
                    </div>
                </div>
                <div class="form-group">
                    <label for="modalExcelDateRange"><span class="label-icon">📅</span> Excel日期位置：</label>
                    <input 
                        type="text" 
                        id="modalExcelDateRange" 
                        v-model="localExcelDateRange" 
                        placeholder="請輸入班表的日期位置範圍" 
                        class="modal-input"
                    />
                    <div class="input-help">
                        <span class="info-icon">ℹ️</span> 請輸入班表中日期所在的範圍，例如：C1~N1
                    </div>
                </div>
                <div class="form-group">
                    <label for="modalVolunteerRowRange"><span class="label-icon">👥</span> 志工位置範圍：</label>
                    <input 
                        type="text" 
                        id="modalVolunteerRowRange" 
                        v-model="localVolunteerRowRange" 
                        placeholder="請輸入志工位置範圍" 
                        class="modal-input"
                    />
                    <div class="input-help">
                        <span class="info-icon">ℹ️</span> 請輸入班表中志工資料的行範圍，例如：6~55
                    </div>
                </div>
                <div class="form-group checkbox-group">
                    <label class="checkbox-label">
                        <input type="checkbox" v-model="localAutoClosePrompt">
                        <span><span class="label-icon">⏱️</span> 自動關閉提示（3秒後）</span>
                    </label>
                    <div class="input-help">
                        <span class="info-icon">ℹ️</span> 啟用後，提示視窗將在3秒後自動關閉
                    </div>
                </div>
            </div>
            <div class="modal-footer">
                <button class="modal-button cancel" @click="$emit('close')">取消</button>
                <button class="modal-button save" @click="saveSettings">儲存設定</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* 模態視窗樣式 */
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
}

.modal-container {
    background-color: white;
    border-radius: 12px;
    width: 90%;
    max-width: 700px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
    overflow: hidden;
    animation: modal-appear 0.3s ease-out;
}

@keyframes modal-appear {
    from {
        opacity: 0;
        transform: translateY(-20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.modal-header {
    background-color: #4299e1;
    color: white;
    padding: 16px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.modal-title {
    margin: 0;
    font-size: 20px;
    font-weight: bold;
    display: flex;
    align-items: center;
}

.settings-icon {
    margin-right: 10px;
}

.close-button {
    background: none;
    border: none;
    color: white;
    font-size: 24px;
    cursor: pointer;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    transition: background-color 0.2s;
}

.close-button:hover {
    background-color: rgba(255, 255, 255, 0.2);
}

.modal-body {
    padding: 20px;
}

.form-group {
    margin-bottom: 20px;
}

.form-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: bold;
    color: #4a5568;
    display: flex;
    align-items: center;
}

.label-icon {
    margin-right: 6px;
}

.modal-input {
    width: 100%;
    padding: 10px 12px;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    font-size: 15px;
    transition: border-color 0.2s;
    box-sizing: border-box;
    overflow: hidden;
    text-overflow: ellipsis;
}

.modal-input:focus {
    border-color: #4299e1;
    outline: none;
    box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.2);
}

.input-help {
    margin-top: 6px;
    font-size: 13px;
    color: #718096;
    line-height: 1.4;
}

.info-icon {
    margin-right: 4px;
}

.checkbox-group {
    display: flex;
    flex-direction: column;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 8px;
    cursor: pointer;
    font-weight: normal;
}

.checkbox-label input[type="checkbox"] {
    width: 18px;
    height: 18px;
    accent-color: #4299e1;
}

.modal-footer {
    padding: 16px 20px;
    background-color: #f8f9fa;
    display: flex;
    justify-content: flex-end;
    gap: 12px;
    border-top: 1px solid #e2e8f0;
}

.modal-button {
    padding: 10px 20px;
    border-radius: 8px;
    font-size: 15px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
}

.cancel {
    background-color: #e2e8f0;
    color: #4a5568;
    border: none;
}

.cancel:hover {
    background-color: #cbd5e0;
}

.save {
    background-color: #4299e1;
    color: white;
    border: none;
}

.save:hover {
    background-color: #3182ce;
}
</style>