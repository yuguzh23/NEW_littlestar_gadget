<script setup>
import { ref, onMounted } from 'vue';
import DownloadExcel from './download excel.vue';
import Setting from './setting.vue';
import SuccessPrompt from './SuccessPrompt.vue';
import * as XLSX from 'xlsx';

// 狀態管理
const file = ref(null); // 存儲檔案物件
const fileName = ref(''); // 存儲檔案名稱
const dates = ref([]); // 存儲日期列表
const selectedDate = ref(''); // 儲存選擇的日期
const yunVolunteers = ref(''); // 儲存雲科場的志工資訊
const linVolunteers = ref(''); // 儲存林內場的志工資訊
const filterText = ref(''); // 篩選文本
const yunCount = ref(0); // 雲科場的志工數量
const linCount = ref(0); // 林內場的志工數量
const isLoading = ref(false); // 是否正在加載數據
const downloadExcelRef = ref(null); // 下載Excel組件的引用
const googleSheetId = ref(''); // Google試算表ID
const showSettingsModal = ref(false); // 控制設定模態視窗的顯示
const autoClosePrompt = ref(true); // 控制提示是否自動關閉
const excelDateRange = ref('C1~N1'); // Excel日期位置範圍
const volunteerRowRange = ref('6~55'); // 志工位置範圍

// Checkbox 狀態
const showName = ref(true); // 是否顯示名字
const showNickname = ref(true); // 是否顯示綽號
const showCode = ref(true); // 是否顯示代號

// 篩選器 Checkbox 狀態
const showYunFilter = ref(false); // 是否顯示雲科場篩選
const showLinFilter = ref(false); // 是否顯示林內場篩選

// Excel 日期轉換函數
const excelDateToJSDate = (excelDate) => {
    return new Date((excelDate - 25569) * 86400 * 1000); // 將Excel日期轉換為JavaScript日期
};

// 格式化日期
const formatDate = (date) => {
    const d = new Date(date);
    const month = d.getMonth() + 1;
    const day = d.getDate();
    return `${month}月${day}日`; // 格式化為 "月日" 的格式
};

// 雲科場代碼列表
const yunCodes = ['動1/Y', '動2/Y', '靜1/Y', '靜2/Y', 'V', 'Y/O', 'Y/C', 'Y/1', 'Y/2', 'Y/3', 'Y/4'];

// 林內場代碼列表
const linCodes = ['動1/L', '動2/L', '靜1/L', '靜2/L', 'VL', 'L/C', 'L/O', 'P', 'L/1', 'L/2', 'L/3', 'L/4'];

// 處理檔案上傳並解析 Excel
const handleFileUpload = (event) => {
    const uploadedFile = event.target.files[0]; // 取得上傳的檔案
    file.value = uploadedFile; // 存儲檔案物件
    fileName.value = uploadedFile.name; // 儲存檔案名稱

    const reader = new FileReader();
    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result); // 讀取檔案內容
        const workbook = XLSX.read(data, { type: 'array' }); // 解析Excel檔案
        const sheetName = workbook.SheetNames[0]; // 取得第一張工作表名稱
        const sheetData = XLSX.utils.sheet_to_json(workbook.Sheets[sheetName], { header: 1 }); // 將工作表轉換為JSON格式

        // 解析日期範圍並格式化
        const dateRange = excelDateRange.value.split('~');
        if (dateRange.length === 2) {
            // 解析起始列位置
            const startCol = dateRange[0].match(/[A-Z]+/)[0];
            const startColIndex = startCol.charCodeAt(0) - 'A'.charCodeAt(0);
            
            // 解析日期並格式化
            dates.value = sheetData[0].slice(startColIndex).map(date => {
                if (!date) return null;
                const jsDate = excelDateToJSDate(date); // 轉換Excel日期為JavaScript日期
                return formatDate(jsDate); // 格式化日期
            }).filter(date => date !== null); // 過濾掉無效日期
        } else {
            // 如果格式不正確，使用默認方式解析
            dates.value = sheetData[0].slice(2).map(date => {
                if (!date) return null;
                const jsDate = excelDateToJSDate(date); // 轉換Excel日期為JavaScript日期
                return formatDate(jsDate); // 格式化日期
            }).filter(date => date !== null); // 過濾掉無效日期
        }
    };
    reader.readAsArrayBuffer(uploadedFile); // 讀取檔案
};

// 處理下載的志工數據
const processVolunteerData = (workbook, sheetData) => {
    // 找到選擇的日期索引
    const dateIndex = dates.value.indexOf(selectedDate.value) + 2;
    
    // 清空志工列表和人數
    yunVolunteers.value = '';
    linVolunteers.value = '';
    yunCount.value = 0;
    linCount.value = 0;
    
    // 解析志工位置範圍
    const rowRange = volunteerRowRange.value.split('~');
    let startRow = 5;
    let endRow = 55;
    
    if (rowRange.length === 2) {
        startRow = parseInt(rowRange[0]) - 1; // 轉為0-based索引
        endRow = parseInt(rowRange[1]);
    }
    
    // 遍歷資料，提取志工資訊
    for (let i = startRow; i < endRow; i++) {
        const row = sheetData[i]; // 取得每一列資料
        if (!row) continue; // 若該行無資料，跳過
        const volunteerName = row[0] || '未命名'; // 志工名字
        const volunteerNickname = row[1] || '無綽號'; // 志工綽號
        let volunteerNameNickname = '';
        
        // 根據選項決定是否顯示名字和綽號
        if (showName.value) {
            volunteerNameNickname += volunteerName;
        }
        if (showNickname.value) {
            volunteerNameNickname += (showName.value ? '(' : '') + volunteerNickname + (showName.value ? ')' : '');
        }
        
        const volunteerCode = row[dateIndex]; // 取得志工代號
        let volunteerInfo = '';
        
        // 合併志工名字、綽號和代號
        if (volunteerNameNickname) {
            volunteerInfo = volunteerNameNickname;
        }
        
        if (showCode.value) {
            volunteerInfo += (volunteerInfo ? ' - ' : '') + volunteerCode;
        }
        
        // 判斷志工代號並將其歸類
        if (yunCodes.includes(volunteerCode)) {
            yunVolunteers.value += `${volunteerInfo}\n`; // 加入雲科場志工列表
            yunCount.value++; // 增加雲科場志工數量
        }
        if (linCodes.includes(volunteerCode)) {
            linVolunteers.value += `${volunteerInfo}\n`; // 加入林內場志工列表
            linCount.value++; // 增加林內場志工數量
        }
    }
    
    // 更新篩選後的志工列表
    updateFilteredVolunteers();
};

// 根據選擇的日期顯示志工值班資訊
const fetchVolunteersByDate = (event) => {
    // 如果是從download-complete事件觸發的，則已經有workbook和sheetData，不需要重新讀取文件
    if (event && event.workbook && event.sheetData) {
        const { workbook, sheetData } = event;
        processVolunteerData(workbook, sheetData);
        return;
    }
    
    // 否則使用原始的讀取文件方式
    if (!file.value || !selectedDate.value) return;
    
    const reader = new FileReader();
    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const sheetName = workbook.SheetNames[0];
        const sheetData = XLSX.utils.sheet_to_json(workbook.Sheets[sheetName], { header: 1 });
        
        // 使用processVolunteerData處理數據
        processVolunteerData(workbook, sheetData);
    };
    reader.readAsArrayBuffer(file.value);
};

// 篩選志工
const filteredVolunteers = ref(''); // 篩選後的志工資訊

// 提示視窗相關狀態
const showSuccessPrompt = ref(false);
const successMessage = ref('');

// 處理設定保存事件
const handleSaveSettings = () => {
    // 顯示儲存成功提示
    successMessage.value = '設定儲存成功';
    showSuccessPrompt.value = true;
    
    // 如果啟用自動關閉，則3秒後自動關閉提示
    if (autoClosePrompt.value) {
        setTimeout(() => {
            showSuccessPrompt.value = false;
        }, 3000);
    }
};

// 更新篩選後的志工列表
const updateFilteredVolunteers = () => {
    const filteredContent = []; // 儲存篩選後的內容

    // 根據選擇的場地篩選志工
    if (showYunFilter.value) {
        filteredContent.push('雲科場：');
        filteredContent.push(...yunVolunteers.value
            .split('\n')
            .filter(volunteer => {
                if (!filterText.value) return true; // 沒有篩選條件時，顯示所有志工
                return !filterText.value.split('\n').some(removeName => volunteer.includes(removeName.trim()));
            }));
    }

    if (showLinFilter.value) {
        filteredContent.push('林內場：');
        filteredContent.push(...linVolunteers.value
            .split('\n')
            .filter(volunteer => {
                if (!filterText.value) return true;
                return !filterText.value.split('\n').some(removeName => volunteer.includes(removeName.trim()));
            }));
    }

    filteredVolunteers.value = filteredContent.join('\n'); // 更新篩選後的志工名單
};

// 自動選擇班表功能
const autoSelectSchedule = async () => {
    // 設置加載狀態
    isLoading.value = true;
    
    try {
        if (!googleSheetId.value) {
            // 如果沒有設置班表ID，則顯示設定模態視窗
            showSettingsModal.value = true;
            throw new Error('請先設置班表ID');
        }
        
        // 使用download excel組件下載班表
        const result = await downloadExcelRef.value.downloadGoogleSheet(googleSheetId.value);
        
        if (!result) {
            throw new Error('下載班表失敗');
        }

        // 處理下載的數據
        const { workbook, sheetData } = result;

        // 設置檔案名稱
        fileName.value = `班表_${new Date().toISOString().split('T')[0]}.xlsx`;
        
        // 將下載的數據轉換為File對象，以便後續可以切換日期
        const excelData = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
        const blob = new Blob([excelData], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        file.value = new File([blob], fileName.value, { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        
        // 解析日期範圍並格式化
        const dateRange = excelDateRange.value.split('~');
        if (dateRange.length === 2) {
            // 解析起始列位置
            const startCol = dateRange[0].match(/[A-Z]+/)[0];
            const startColIndex = startCol.charCodeAt(0) - 'A'.charCodeAt(0);
            
            // 解析日期並格式化
            dates.value = sheetData[0].slice(startColIndex).map(date => {
                if (!date) return null;
                const jsDate = excelDateToJSDate(date); // 轉換Excel日期為JavaScript日期
                return formatDate(jsDate); // 格式化日期
            }).filter(date => date !== null); // 過濾掉無效日期
        } else {
            // 如果格式不正確，使用默認方式解析
            dates.value = sheetData[0].slice(2).map(date => {
                if (!date) return null;
                const jsDate = excelDateToJSDate(date); // 轉換Excel日期為JavaScript日期
                return formatDate(jsDate); // 格式化日期
            }).filter(date => date !== null); // 過濾掉無效日期
        }
        
        // 自動選擇今天的日期或第一個可用日期
        const today = new Date();
        const month = today.getMonth() + 1;
        const day = today.getDate();
        const formattedDate = `${month}月${day}日`;
        
        // 如果日期列表中有今天的日期，則自動選擇
        if (dates.value.includes(formattedDate)) {
            selectedDate.value = formattedDate;
            // 使用下載的數據處理志工資訊
            processVolunteerData(workbook, sheetData);
        } else if (dates.value.length > 0) {
            // 如果沒有今天的日期，則選擇第一個可用日期
            selectedDate.value = dates.value[0];
            // 使用下載的數據處理志工資訊
            processVolunteerData(workbook, sheetData);
        } else {
            alert('無法自動選擇班表，班表格式可能不正確');
        }
    } catch (error) {
        console.error('自動選擇班表失敗:', error);
        alert(`自動選擇班表失敗: ${error.message}`);
    } finally {
        // 無論成功或失敗，都關閉加載狀態
        isLoading.value = false;
    }
};


</script>

<template>
    <div class="app-container">
        <!-- 引入下載Excel組件 -->
        <DownloadExcel ref="downloadExcelRef" @download-complete="fetchVolunteersByDate" @download-error="error => alert(error)" :autoClosePrompt="autoClosePrompt" />
        <!-- 頂部藍色標題區域 -->
        <div class="header">
            <div class="header-content">
                <h1 class="title">
                    <span class="star-icon">⭐</span> 小星星 小工具 <span class="version">V3.0</span>
                    <button class="settings-button" @click="showSettingsModal = true">
                        <span class="settings-icon">⚙️</span>
                    </button>
                </h1>
                <div class="subtitle">線上版</div>
            </div>
        </div>
        
        <!-- 設定模態視窗 -->
        <div v-if="showSettingsModal">
            <Setting 
                :googleSheetId="googleSheetId"
                :excelDateRange="excelDateRange"
                :volunteerRowRange="volunteerRowRange"
                :autoClosePrompt="autoClosePrompt"
                @update:googleSheetId="googleSheetId = $event"
                @update:excelDateRange="excelDateRange = $event"
                @update:volunteerRowRange="volunteerRowRange = $event"
                @update:autoClosePrompt="autoClosePrompt = $event"
                @save-settings="handleSaveSettings(); showSettingsModal = false;"
                @close="showSettingsModal = false"
            />
        </div>
        
        <!-- 成功提示視窗 -->
        <div class="success-prompt-overlay" v-if="showSuccessPrompt" @click="showSuccessPrompt = false">
            <div class="success-prompt-container" @click.stop>
                <div class="success-prompt-header">
                    <h2 class="success-prompt-title"><span class="success-icon">✅</span> 操作成功</h2>
                    <button class="close-button" @click="showSuccessPrompt = false">×</button>
                </div>
                <div class="success-prompt-body">
                    <p class="success-message">{{ successMessage }}</p>
                </div>
                <div class="success-prompt-footer">
                    <button class="modal-button save" @click="showSuccessPrompt = false">確定</button>
                </div>
            </div>
        </div>

        <!-- 主要內容區域 -->
        <div class="page-container">
            <!-- 班表按鈕區域 - 包含自動選擇和手動上傳按鈕 -->
            <div class="button-container">
                <button class="upload-button auto-button" @click="autoSelectSchedule" :disabled="isLoading">
                    <span class="button-icon">🔄</span> 自動選擇班表
                </button>
                <button class="upload-button" @click="$refs.fileInput.click()">
                    <span class="button-icon">📄</span> 手動上傳班表
                </button>
                <input type="file" ref="fileInput" @change="handleFileUpload" class="hidden-input" />
                <div v-if="fileName" class="file-name">
                    <span class="file-icon">📋</span> {{ fileName }}
                </div>
                <div v-if="isLoading" class="loading-indicator">
                    <span class="loading-icon">⏳</span> 正在載入班表...
                </div>
            </div>

            <div class="content-container">
                <!-- 左側 - 班表查看區域 -->
                <div class="section schedule-section">
                    <h2 class="section-title">
                        <span class="section-icon">📅</span> 班表查看
                    </h2>
                    <div class="date-selection">
                        <label><span class="label-icon">📆</span> 選擇日期：</label>
                        <select v-model="selectedDate" @change="fetchVolunteersByDate()" class="date-dropdown">
                            <option value="" disabled>--請選擇日期--</option>
                            <option v-for="date in dates" :key="date" :value="date">{{ date }}</option>
                        </select>
                    </div>

                    <div class="display-options">
                        <span class="display-label"><span class="label-icon">👁️</span> 顯示選項：</span>
                        <label class="checkbox-label">
                            <input type="checkbox" v-model="showName" @change="fetchVolunteersByDate">
                            <span>名字</span>
                        </label>
                        <label class="checkbox-label">
                            <input type="checkbox" v-model="showNickname" @change="fetchVolunteersByDate">
                            <span>綽號</span>
                        </label>
                        <label class="checkbox-label">
                            <input type="checkbox" v-model="showCode" @change="fetchVolunteersByDate">
                            <span>代號</span>
                        </label>
                    </div>

                    <!-- 改為左右並排 -->
                    <div class="volunteer-lists-horizontal">
                        <div class="volunteer-section">
                            <h3 class="venue-title">
                                <span class="venue-icon">🏫</span> 雲科場：({{ yunCount }}人)
                            </h3>
                            <textarea 
                                v-model="yunVolunteers" 
                                readonly 
                                class="volunteer-list"
                            ></textarea>
                        </div>

                        <div class="volunteer-section">
                            <h3 class="venue-title">
                                <span class="venue-icon">🏕️</span> 林內場：({{ linCount }}人)
                            </h3>
                            <textarea 
                                v-model="linVolunteers" 
                                readonly 
                                class="volunteer-list"
                            ></textarea>
                        </div>
                    </div>
                </div>

                <!-- 右側 - 志工篩選區域 -->
                <div class="section filter-section">
                    <h2 class="section-title">
                        <span class="section-icon">🔍</span> 志工篩選
                    </h2>
                    <div class="filter-description">
                        <span class="info-icon">ℹ️</span> 功能說明：輸入志工姓名或綽號，篩選出剩餘的志工名單。
                    </div>

                    <div class="venue-filters">
                        <label class="venue-label"><span class="label-icon">🏢</span> 場地選擇：</label>
                        <div class="venue-options">
                            <label class="checkbox-label">
                                <input type="checkbox" v-model="showYunFilter" @change="updateFilteredVolunteers">
                                <span>雲科場</span>
                            </label>
                            <label class="checkbox-label">
                                <input type="checkbox" v-model="showLinFilter" @change="updateFilteredVolunteers">
                                <span>林內場</span>
                            </label>
                        </div>
                    </div>

                    <!-- 搜尋和結果左右並排 -->
                    <div class="filter-content-horizontal">
                        <div class="search-section">
                            <h3 class="search-title">
                                <span class="label-icon">🔎</span> 篩選志工： 
                            </h3>
                            <textarea 
                                v-model="filterText" 
                                @input="updateFilteredVolunteers"
                                class="search-input"
                                placeholder="請輸入志工姓名或綽號..."
                            ></textarea>
                        </div>

                        <div class="results-section">
                            <h3 class="results-title">
                                <span class="label-icon">👥</span> 志工列表：
                            </h3>
                            <textarea 
                                v-model="filteredVolunteers" 
                                readonly 
                                class="filtered-results"
                            ></textarea>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* 全域樣式設定 */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

/* 確保整個頁面沒有邊距以實現完美置中 */
html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #f0f2f5;
}

/* 確保Vue應用容器也是全寬並置中 */
#app {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

/* 置中設計 - 整個應用容器置中 */
.app-container {
    font-family: 'Microsoft JhengHei', Arial, sans-serif;
    display: flex;
    flex-direction: column;
    background-color: #f8f9fa;
    border-radius: 30px;
    max-width: 1500px; /* 設定最大寬度 */
    width: 100%; /* 確保全寬 */
    margin: 20px auto; /* 整個應用容器水平置中，上下間距 */
    min-height: calc(100vh - 40px); /* 留出上下邊距 */
    box-shadow: 0 0 30px rgba(0, 0, 0, 0.1); /* 增加陰影提升視覺層次 */
}

.header {
    background: linear-gradient(135deg, #6ab7ff 0%, #4299e1 100%);
    padding: 20px 0;
    color: white;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border-radius: 30px 30px 30px 30px;
    width: 100%; /* 確保標題區域佔滿容器寬度 */
}

.header-content {
    max-width: 1400px;
    margin: 0 auto;
    padding: 0 20px;
    text-align: center;
}

.title {
    margin: 0;
    font-size: 32px;
    font-weight: bold;
    letter-spacing: 1px;
}

.star-icon {
    font-size: 36px;
    vertical-align: middle;
    margin-right: 8px;
}

.version {
    font-size: 18px;
    background-color: rgba(255, 255, 255, 0.2);
    padding: 2px 8px;
    border-radius: 12px;
    margin-left: 8px;
}

.subtitle {
    margin-top: 8px;
    font-size: 16px;
    opacity: 0.9;
}

/* 主要內容區域置中 */
.page-container {
    flex: 1;
    max-width: 1400px;
    width: 100%;
    margin: 0 auto;
    padding: 0 20px; /* 增加水平內邊距 */
    display: flex;
    flex-direction: column;
    align-items: center; /* 子元素水平置中 */
}

/* 手動上傳班表按鈕區域 */
.button-container {
    margin: 15px 0;
    display: flex;
    align-items: center;
    gap: 15px;
    width: 100%;
    max-width: 1400px;
}

/* Google試算表ID輸入框 */
.sheet-id-input {
    display: flex;
    align-items: center;
    background-color: #f0f4f8;
    padding: 8px 12px;
    border-radius: 8px;
    margin-right: 10px;
    flex-grow: 1;
    max-width: 400px;
}

.sheet-id-field {
    flex-grow: 1;
    border: 1px solid #d0d7de;
    border-radius: 4px;
    padding: 8px 12px;
    font-size: 14px;
    margin-left: 8px;
    background-color: white;
    transition: border-color 0.3s;
}

.sheet-id-field:focus {
    outline: none;
    border-color: #4299e1;
    box-shadow: 0 0 0 2px rgba(66, 153, 225, 0.2);
}

.upload-button {
    background: linear-gradient(to right, #ff7e5f, #feb47b);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 50px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    font-size: 16px;
    font-weight: bold;
    box-shadow: 0 4px 10px rgba(255, 126, 95, 0.3);
    transition: transform 0.2s, box-shadow 0.2s;
    margin-right: 10px;
}

.upload-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(255, 126, 95, 0.4);
}

.upload-button:disabled {
    background: linear-gradient(to right, #ccc, #ddd);
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
}

.auto-button {
    background: linear-gradient(to right, #4facfe, #00f2fe);
    box-shadow: 0 4px 10px rgba(79, 172, 254, 0.3);
}

.auto-button:hover {
    box-shadow: 0 6px 12px rgba(79, 172, 254, 0.4);
}

.loading-indicator {
    display: flex;
    align-items: center;
    padding: 8px 16px;
    background-color: #e6f7ff;
    border-radius: 8px;
    color: #1890ff;
    font-size: 15px;
    font-weight: 500;
    animation: pulse 1.5s infinite;
}

@keyframes pulse {
    0% { opacity: 0.6; }
    50% { opacity: 1; }
    100% { opacity: 0.6; }
}

.loading-icon {
    margin-right: 8px;
    font-size: 16px;
    animation: spin 2s linear infinite;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

.button-icon {
    margin-right: 8px;
    font-size: 20px;
}

.hidden-input {
    display: none;
}

/* 檔名顯示樣式 */
.file-name {
    display: flex;
    align-items: center;
    padding: 8px 16px;
    background-color: #edf2f7;
    border-radius: 8px;
    color: #4a5568;
    font-size: 15px;
    font-weight: 500;
    max-width: 800px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateX(-10px);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

.file-icon {
    margin-right: 8px;
    font-size: 16px;
}

/* 主要內容區域 - 左右面板置中 */
.content-container {
    display: flex;
    gap: 20px;
    margin-bottom: 20px;
    width: 100%;
    max-width: 1400px;
}

.section {
    background-color: white;
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
    transition: transform 0.3s, box-shadow 0.3s;
}

/* 調整左右面板比例為相等 */
.schedule-section, .filter-section {
    flex: 1; /* 左右面板等寬 */
}

.section:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
}

.section-title {
    margin-top: 0;
    margin-bottom: 15px;
    font-size: 22px;
    color: #2d3748;
    border-bottom: 2px solid #e2e8f0;
    padding-bottom: 10px;
    display: flex;
    align-items: center;
}

.section-icon {
    margin-right: 10px;
    font-size: 24px;
}

.date-selection {
    margin-bottom: 20px;
    display: flex;
    align-items: center;
}

.date-dropdown {
    padding: 10px 16px;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    width: 220px;
    font-size: 15px;
    margin-left: 10px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
}

.display-options {
    display: flex;
    margin-bottom: 20px;
    gap: 20px;
    align-items: center;
    background-color: #f8f9fa;
    padding: 12px 16px;
    border-radius: 10px;
}

.display-label {
    font-weight: bold;
    color: #4a5568;
    margin-right: 10px;
    display: flex;
    align-items: center;
}

.label-icon {
    margin-right: 6px;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    padding: 6px 10px;
    border-radius: 6px;
    transition: background-color 0.2s;
}

.checkbox-label:hover {
    background-color: #edf2f7;
}

.checkbox-label input[type="checkbox"] {
    width: 16px;
    height: 16px;
    accent-color: #4299e1;
}

/* 志工列表左右並排 */
.volunteer-lists-horizontal {
    display: flex;
    gap: 20px;
}

.volunteer-section {
    margin-bottom: 10px;
    flex: 1;
}

.venue-title {
    margin: 0 0 10px 0;
    font-size: 18px;
    font-weight: bold;
    color: #4a5568;
    display: flex;
    align-items: center;
}

.venue-icon {
    margin-right: 8px;
}

.volunteer-list {
    width: 100%;
    height: 350px;
    padding: 14px;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    resize: none;
    font-size: 15px;
    line-height: 1.6;
    background-color: #f8fafc;
}

.filter-description {
    background-color: #ebf8ff;
    padding: 10px 15px;
    border-radius: 10px;
    margin-bottom: 15px;
    font-size: 14px;
    line-height: 1.6;
    color: #2c5282;
    border-left: 4px solid #4299e1;
    display: flex;
    align-items: flex-start;
}

.info-icon {
    margin-right: 8px;
    font-size: 18px;
    flex-shrink: 0;
}

.venue-filters {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
    background-color: #f8f9fa;
    padding: 10px 16px;
    border-radius: 10px;
}

.venue-label {
    margin-right: 15px;
    font-weight: bold;
    color: #4a5568;
    display: flex;
    align-items: center;
}

.venue-options {
    display: flex;
    gap: 20px;
}

/* 篩選區域左右並排 */
.filter-content-horizontal {
    display: flex;
    gap: 20px;
}

.search-section, .results-section {
    flex: 1;
    margin-bottom: 10px;
}

.search-title, .results-title {
    margin: 0 0 10px 0;
    font-size: 18px;
    font-weight: bold;
    color: #4a5568;
    display: flex;
    align-items: center;
}

.search-input {
    width: 100%;
    height: 350px;
    padding: 14px;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    resize: none;
    font-size: 15px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
}

.filtered-results {
    width: 100%;
    height: 350px;
    padding: 14px;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    resize: none;
    font-size: 15px;
    line-height: 1.6;
    background-color: #f8fafc;
}

/* 設定按鈕樣式 */
.settings-button {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 24px;
    margin-left: 15px;
    padding: 5px;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    transition: all 0.3s ease;
    background-color: rgba(255, 255, 255, 0.2);
    vertical-align: middle;
}

.settings-button:hover {
    background-color: rgba(255, 255, 255, 0.3);
    transform: rotate(30deg);
}

.settings-icon {
    font-size: 20px;
}

/* 模態視窗樣式 */
.modal-overlay, .success-prompt-overlay {
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

.modal-container, .success-prompt-container {
    background-color: white;
    border-radius: 16px;
    width: 90%;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
    animation: slideIn 0.3s ease;
}

.modal-container {
    max-width: 600px; /* 增加寬度 */
}

.success-prompt-container {
    max-width: 400px;
}

@keyframes slideIn {
    from { transform: translateY(-50px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}

.success-icon {
    color: #48bb78;
    margin-right: 8px;
}

.success-message {
    font-size: 16px;
    color: #2d3748;
    text-align: center;
}

.modal-header, .success-prompt-header {
    padding: 15px 20px;
    border-bottom: 1px solid #e2e8f0;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.modal-title, .success-prompt-title {
    margin: 0;
    font-size: 20px;
    color: #2d3748;
    display: flex;
    align-items: center;
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

.modal-body, .success-prompt-body {
    padding: 20px;
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: 600;
    color: #4a5568;
}

.modal-input {
    width: 100%;
    padding: 12px 15px;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    font-size: 16px;
    transition: border-color 0.3s, box-shadow 0.3s;
}

.modal-input:focus {
    outline: none;
    border-color: #4299e1;
    box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.2);
}

.input-help {
    margin-top: 8px;
    font-size: 14px;
    color: #718096;
    line-height: 1.5;
}

.input-help strong {
    color: #4299e1;
    background-color: #ebf8ff;
    padding: 2px 4px;
    border-radius: 4px;
}

.modal-footer, .success-prompt-footer {
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

.modal-button.cancel {
    background-color: #edf2f7;
    color: #4a5568;
    border: 1px solid #e2e8f0;
}

.modal-button.cancel:hover {
    background-color: #e2e8f0;
}

.modal-button.save {
    background: linear-gradient(to right, #4facfe, #00f2fe);
    color: white;
    border: none;
    box-shadow: 0 4px 6px rgba(66, 153, 225, 0.3);
}

.modal-button.save:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 8px rgba(66, 153, 225, 0.4);
}

/* 響應式設計 */
@media (max-width: 1600px) {
    .app-container {
        max-width: 95%;
    }
    
    .page-container, .header-content {
        max-width: 95%;
    }
}

@media (max-width: 1200px) {
    .app-container {
        max-width: 100%;
        border-radius: 0;
        margin: 0; /* 移除邊距 */
        min-height: 100vh; /* 全高 */
    }
    
    .page-container {
        max-width: 95%;
        padding: 15px 20px;
    }
    
    .content-container {
        gap: 20px;
    }
}

@media (max-width: 900px) {
    .content-container {
        flex-direction: column;
    }
    
    .section {
        margin-bottom: 20px;
        width: 100%;
    }
    
    /* 在小螢幕上改回垂直排列 */
    .volunteer-lists-horizontal,
    .filter-content-horizontal {
        flex-direction: column;
    }
    
    .volunteer-list,
    .search-input,
    .filtered-results {
        height: 250px; /* 在小屏幕上調整高度 */
    }
}

@media (max-width: 600px) {
    .title {
        font-size: 24px;
    }
    
    .star-icon {
        font-size: 24px;
    }
    
    .display-options {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
    
    .date-selection {
        flex-direction: column;
        align-items: flex-start;
        gap: 8px;
    }
    
    .date-dropdown {
        width: 100%;
        margin-left: 0;
    }
    
    .venue-filters {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
    
    .button-container {
        flex-direction: column;
        align-items: flex-start;
    }
    
    .file-name {
        max-width: 100%;
    }
    
    .volunteer-list,
    .search-input,
    .filtered-results {
        height: 200px; /* 在超小屏幕上進一步調整高度 */
    }
}
</style>