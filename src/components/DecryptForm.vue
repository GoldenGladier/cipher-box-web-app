<template>
  <div class="decrypt">
    <div class="container">
      <h2>Decrypt Files</h2>

      <div class="mode-selection">
        <label>
          <input type="radio" name="mode" value="folder" v-model="selectedMode" />
          Folder
        </label>
        <label>
          <input type="radio" name="mode" value="files" v-model="selectedMode" />
          Files
        </label>
      </div>

      <div v-if="selectedMode === 'files'" class="file-input">
        <label for="files">Select encrypted files:</label>
        <input type="file" id="files" multiple @change="onFilesSelected" />
      </div>

      <div v-else-if="selectedMode === 'folder'" class="file-input">
        <label for="folder">Select encrypted folder:</label>
        <input type="file" id="folder" webkitdirectory @change="onFolderSelected" />
      </div>

      <div class="password-input">
        <label for="password">Enter decryption password:</label>
        <input type="password" id="password" v-model="password" />
      </div>

      <div class="actions">
        <button @click="decryptFiles">Decrypt</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DecryptForm',
  data() {
    return {
      selectedMode: 'folder', // Modo predeterminado
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
    async decryptFiles() {
      try {
        // Convertir la contraseña en una clave criptográfica
        const key = await this.generateKey(this.password);

        // Desencriptar archivos o carpeta
        if (this.selectedMode === 'files' && this.files.length > 0) {
          for (let file of this.files) {
            const arrayBuffer = await file.arrayBuffer();
            const decryptedData = await this.decryptData(arrayBuffer, key);
            this.downloadDecryptedFile(decryptedData, file.name);
          }
        } else if (this.selectedMode === 'folder' && this.folder.length > 0) {
          for (let file of this.folder) {
            const arrayBuffer = await file.arrayBuffer();
            const decryptedData = await this.decryptData(arrayBuffer, key);
            this.downloadDecryptedFile(decryptedData, file.name);
          }
        } else {
          alert('Please select encrypted files or a folder to decrypt.');
        }
      } catch (error) {
        console.error('Decryption failed:', error);
        alert('Decryption failed. Please check the password and try again.');
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
          salt: encoder.encode('salt'), // Usar una sal apropiada en una aplicación real
          iterations: 100000,
          hash: 'SHA-256'
        },
        keyMaterial,
        { name: 'AES-GCM', length: 256 },
        false,
        ['decrypt']
      );
    },
    async decryptData(data, key) {
      try {
        // Extraer el IV del inicio de los datos
        const iv = data.slice(0, 12);
        const encryptedData = data.slice(12);

        const decrypted = await window.crypto.subtle.decrypt(
          {
            name: 'AES-GCM',
            iv: iv,
          },
          key,
          encryptedData
        );

        return decrypted;
      } catch (error) {
        console.error('Error during decryption:', error);
        throw new Error('Decryption failed');
      }
    },
    downloadDecryptedFile(decryptedData, fileName) {
      const blob = new Blob([decryptedData], { type: 'application/octet-stream' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = fileName.replace('.encrypted', ''); // Remover la extensión .encrypted
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
}

button:hover {
  background-color: #0056b3;
}
</style>
