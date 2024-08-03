<template>
    <div>
      <h2>Formulario</h2>
      <form @submit.prevent="handleSubmit">
        <div class="mb-3">
          <label for="nombre" class="form-label">Nombre</label>
          <input type="text" class="form-control" id="nombre" v-model="nombre" />
        </div>
        <div class="mb-3">
          <label for="apellido" class="form-label">Apellido</label>
          <input type="text" class="form-control" id="apellido" v-model="apellido" required/>
        </div>
        <div class="mb-3">
          <label for="cedula" class="form-label">Número de Cédula</label>
          <input type="text" class="form-control" id="cedula" v-model="cedula" required />
        </div>
        <div class="mb-3">
          <label for="correo" class="form-label">Correo</label>
          <input type="email" class="form-control" id="correo" v-model="correo" />
        </div>
        <div class="mb-3">
          <label for="foto" class="form-label">Foto</label>
          <input type="file" class="form-control" id="foto" @change="handleFileChange" ref="fotoInput" />
        </div>
        <button type="submit" class="btn btn-primary">Agregar</button>
      </form>
    </div>
  </template>
  
  <script setup>
  import { ref, defineProps } from 'vue';
  import axios from 'axios';
  import Swal from 'sweetalert2';
  
  const nombre = ref('');
  const apellido = ref('');
  const cedula = ref('');
  const correo = ref('');
  const foto = ref(null);
  
  const fotoInput = ref(null);

  const emit = defineEmits(['addRecord'])

  AWS.config.update({
    accessKeyId: import.meta.env.VITE_ACCESS_KEY_ID,       // Reemplaza con tu Access Key ID
    secretAccessKey: import.meta.env.VITE_SECRET_ACCESS_KEY, // Reemplaza con tu Secret Access Key
    sessionToken: import.meta.env.VITE_SESSION_TOKEN,
    region:'us-east-1'                    // Reemplaza con la región de tu bucket
  });

  const s3 = new AWS.S3();

  const handleFileChange = (event) => {
    foto.value = event.target.files[0];
  };

  const mensaje = (icon,title) => {
    const Toast = Swal.mixin({
      toast: true,
      position: "top-end",
      showConfirmButton: false,
      timer: 3000,
      timerProgressBar: true,
      didOpen: (toast) => {
        toast.onmouseenter = Swal.stopTimer;
        toast.onmouseleave = Swal.resumeTimer;
      }
    });
    Toast.fire({
      icon,
      title
    });
  }

  const  subirImagenABucket = (file) => {
  const params = {
    Bucket: 'gponce-taller-dynamo-lambda-s3', // Reemplaza con el nombre de tu bucket
    Key: file.name,                // El nombre del archivo en S3
    Body: file,                    // El archivo a subir
    ContentType: file.type,        // El tipo de contenido del archivo
    ACL: 'public-read'             // Permisos de lectura pública
  };

  s3.upload(params, function(err, data) {

    if (err) {
      console.log('Error al subir la imagen:', err);
    } 
    if(data){
      console.log('Imagen subida correctamente:', data);
      console.log('URL de la imagen:', data.Location);
      submitData(data.Location)
    }
  });
}
  
  const handleSubmit = async () => {
    if (foto.value) {
      subirImagenABucket(foto.value)
    }else{
      submitData(null)
    }
  };

  const generarUUID = () => {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
          const r = Math.random() * 16 | 0;
          const v = c === 'x' ? r : (r & 0x3 | 0x8);
          return v.toString(16);
      });
  }
  
  const submitData = async (imageUrl) => {
    const record = {
      id: generarUUID(),
      nombre: nombre.value,
      apellido: apellido.value,
      cedula: cedula.value,
      correo: correo.value,
      imageUrl,
    };
  
    const options = {
      method: 'POST',
      url: 'https://eh0164rwce.execute-api.us-east-1.amazonaws.com/default/taller-lamda',
      headers: {'content-type': 'application/json'},
      data: {
        TableName: 'taller-formulario',
        Item: record
      }
    };

    try {
      const { data } = await axios.request(options);
      console.log(data);
      mensaje("success","Registro exitoso");
      resetForm();
      emit("addRecord")
    } catch (error) {
      console.error(error);
      mensaje("error",error);
    }
  };
  
  const resetForm = () => {
    nombre.value = '';
    apellido.value = '';
    cedula.value = '';
    correo.value = '';
    foto.value = null;
    fotoInput.value.value = '';
  };
  </script>
  