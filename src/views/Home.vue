<template>
  <div class="home">
    <div class="container">
      <div class="tabs">
        <button 
          :class="{ active: activeTab === 'encrypt' }" 
          @click="activeTab = 'encrypt'"
        >
          Encrypt
        </button>
        <button 
          :class="{ active: activeTab === 'decrypt' }" 
          @click="activeTab = 'decrypt'"
        >
          Decrypt
        </button>
      </div>
      <div class="tab-content">
        <EncryptForm v-if="activeTab === 'encrypt'" />
        <DecryptForm v-if="activeTab === 'decrypt'" />
      </div>
      <div class="container">
        <TestFiles />
      </div>
    </div>
  </div>
</template>

<script>
import EncryptForm from '@/components/EncryptForm.vue';
import DecryptForm from '@/components/DecryptForm.vue';
import TestFiles from '@/components/TestFiles.vue';

export default {
  name: 'Home',
  components: {
    EncryptForm,
    DecryptForm,
    TestFiles,
  },
  data() {
    return {
      activeTab: 'encrypt', // Default to the encrypt tab
    };
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.tabs {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.tabs button {
  padding: 10px 20px;
  font-size: 16px;
  color: #333;
  background-color: #f8f9fa;
  border: 1px solid #ddd;
  cursor: pointer;
  transition: background-color 0.3s;
}

.tabs button.active {
  background-color: #007bff;
  color: #fff;
}

.tabs button:not(.active):hover {
  background-color: #e2e6ea;
}

.tab-content {
  margin-top: 20px;
}
</style>
