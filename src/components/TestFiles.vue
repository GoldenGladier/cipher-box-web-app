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
            try {
                const [fileHandle] = await window.showOpenFilePicker();
                const file = await fileHandle.getFile();
                const contents = await file.text();
                alert("Archivo abierto exitosamente: " + file.name);
                console.log(contents);
            } catch (error) {
                alert("Error al abrir archivo: " + error.message);
                console.error("Error al abrir archivo:", error);
            }
        },
        async saveFile() {
            try {
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
                alert("Archivo guardado exitosamente");
            } catch (error) {
                alert("Error al guardar archivo: " + error.message);
                console.error("Error al guardar archivo:", error);
            }
        }
    }
}
</script>