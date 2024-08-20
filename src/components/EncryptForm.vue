<template>
  <div class="home">
    <div class="container">
      <h2>Select Mode</h2>
      <div class="mode-selection">
        <label>
          <input type="radio" name="mode" value="folder" v-model="selectedMode" />
          Folder test
        </label>
        <label>
          <input type="radio" name="mode" value="files" v-model="selectedMode" />
          Files
        </label>
      </div>

      <div v-if="selectedMode === 'files'" class="file-input">
        <label for="files">Select files:</label>
        <input type="file" id="files" multiple @change="onFilesSelected" />
      </div>

      <div v-else-if="selectedMode === 'folder'" class="file-input">
        <label for="folder">Select folder:</label>
        <input type="file" id="folder" webkitdirectory @change="onFolderSelected" />
      </div>

      <div class="password-input">
        <label for="password">Enter password:</label>
        <input type="password" id="password" v-model="password" />
      </div>

      <div class="actions">
        <button @click="encryptFiles">Encrypt</button>
      </div>
    </div>
  </div>
</template>

<script>
import JSZip from 'jszip';
import { saveAs } from 'file-saver';

export default {
  name: 'EncryptForm',
  data() {
    return {
      selectedMode: 'folder', // Default mode is folder
      password: '',
      files: [],
      folder: null,
    };
  },
  methods: {
    onFilesSelected(event) {
      this.files = event.target.files;
    },
    onFolderSelected(event) {
      this.folder = event.target.files;
    },
    async encryptFiles() {
      try {
        // Convert password to a crypto key
        const key = await this.generateKey(this.password);
        
        const fileArray = this.selectedMode === 'files' ? this.files : this.folder;

        if (fileArray.length === 0) {
          alert('Please select files or a folder to encrypt.');
          return;
        }

        // Check if we need to create a zip file
        if (this.selectedMode === 'folder' || fileArray.length > 8) {
          const zip = new JSZip();
          for (let file of fileArray) {
            const arrayBuffer = await file.arrayBuffer();
            const encryptedData = await this.encryptData(arrayBuffer, key);
            zip.file(`${file.name}.encrypted`, new Uint8Array(encryptedData));
          }
          const content = await zip.generateAsync({ type: 'blob' });
          saveAs(content, 'encrypted_files.zip');
        } else {
          // Download each file individually
          for (let file of fileArray) {
            const arrayBuffer = await file.arrayBuffer();
            const encryptedData = await this.encryptData(arrayBuffer, key);
            this.downloadEncryptedFile(encryptedData, file.name);
          }
        }
      } catch (error) {
        console.error('Encryption failed:', error);
      }
    },
    async generateKey(password) {
      const encoder = new TextEncoder();
      const keyMaterial = await window.crypto.subtle.importKey(
        'raw',
        encoder.encode(password),
        { name: 'PBKDF2' },
        false,
        ['deriveKey']
      );

      return window.crypto.subtle.deriveKey(
        {
          name: 'PBKDF2',
          salt: encoder.encode('salt'), // Use a proper salt in a real application
          iterations: 100000,
          hash: 'SHA-256'
        },
        keyMaterial,
        { name: 'AES-GCM', length: 256 },
        false,
        ['encrypt', 'decrypt']
      );
    },
    async encryptData(data, key) {
      const iv = window.crypto.getRandomValues(new Uint8Array(12)); // Generate a random IV
      const encrypted = await window.crypto.subtle.encrypt(
        {
          name: 'AES-GCM',
          iv: iv,
        },
        key,
        data
      );
      return new Uint8Array([...iv, ...new Uint8Array(encrypted)]); // Concatenate IV and encrypted data
    },
    downloadEncryptedFile(encryptedData, fileName) {
      const blob = new Blob([encryptedData], { type: 'application/octet-stream' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `${fileName}.encrypted`;
      a.click();
      URL.revokeObjectURL(url);
    },
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

h2 {
  margin-bottom: 20px;
  font-size: 24px;
  color: #333;
}

.mode-selection {
  margin-bottom: 20px;
}

.mode-selection label {
  margin-right: 15px;
  font-size: 18px;
  color: #555;
}

.file-input,
.password-input {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-size: 16px;
  color: #555;
}

input[type="file"],
input[type="password"] {
  width: calc(100% - 10px);
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.actions {
  margin-top: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

button:hover {
  background-color: #0056b3;
}
</style>
