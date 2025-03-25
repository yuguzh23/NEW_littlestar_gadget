<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
    modelValue: {
        type: Boolean,
        default: false
    },
    message: {
        type: String,
        default: ''
    }
});

const emit = defineEmits(['update:modelValue']);
</script>

<template>
    <!-- 成功提示視窗 -->
    <div class="success-prompt-overlay" @click="$emit('update:modelValue', false)">
        <div class="success-prompt-container" @click.stop>
            <div class="success-prompt-header">
                <h2 class="success-prompt-title"><span class="success-icon">✅</span> 操作成功</h2>
                <button class="close-button" @click="$emit('update:modelValue', false)">×</button>
            </div>
            <div class="success-prompt-body">
                <p class="success-message">{{ message }}</p>
            </div>
            <div class="success-prompt-footer">
                <button class="modal-button save" @click="$emit('update:modelValue', false)">確定</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.success-prompt-overlay {
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

.success-prompt-container {
    background-color: white;
    border-radius: 12px;
    width: 90%;
    max-width: 400px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
    overflow: hidden;
    animation: prompt-appear 0.3s ease-out;
}

@keyframes prompt-appear {
    from {
        opacity: 0;
        transform: translateY(-20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.success-prompt-header {
    background-color: #48bb78;
    color: white;
    padding: 16px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.success-prompt-title {
    margin: 0;
    font-size: 20px;
    font-weight: bold;
    display: flex;
    align-items: center;
}

.success-icon {
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

.success-prompt-body {
    padding: 20px;
}

.success-message {
    margin: 0;
    font-size: 16px;
    color: #4a5568;
    text-align: center;
}

.success-prompt-footer {
    padding: 16px 20px;
    background-color: #f8f9fa;
    display: flex;
    justify-content: center;
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

.save {
    background-color: #48bb78;
    color: white;
    border: none;
}

.save:hover {
    background-color: #38a169;
}
</style>