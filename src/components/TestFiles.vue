<template>
  <div>
    <button @click="openFile">Abrir Archivo</button>
    <button @click="saveFile">Guardar Archivo</button>
  </div>
</template>

<script>
export default {
    component: 'TestFiles',
    methods: {
        async openFile() {
        // Permitir al usuario seleccionar un archivo
        const [fileHandle] = await window.showOpenFilePicker();
        const file = await fileHandle.getFile();
        const contents = await file.text();
        console.log(contents);
        },
        async saveFile() {
        // Permitir al usuario seleccionar una carpeta y guardar un archivo en ella
        const options = {
            types: [{
            description: 'Text Files',
            accept: {
                'text/plain': ['.txt'],
            },
            }],
        };
        const handle = await window.showSaveFilePicker(options);
        const writableStream = await handle.createWritable();
        await writableStream.write("Contenido de ejemplo");
        await writableStream.close();
        }
    }
}
</script>