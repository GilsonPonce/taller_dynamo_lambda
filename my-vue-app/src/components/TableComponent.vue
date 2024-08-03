<template>
    <div>
      <h2>Registros</h2>
      <div class="row row-cols-2 row-cols-md-2 g-4">
        <div class="col">
          <div v-for="(record,index) in records" :key="record.id" class="card" style="width: 18rem;">
            <img :src="record.imageUrl" class="card-img-top" >
            <div class="card-body">
              <h5 class="card-title">{{ record.correo }}</h5>
              <p class="card-text">{{ `${record.nombre} ${record.apellido}` }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { defineProps } from 'vue';
  
  import 'bootstrap/dist/css/bootstrap.css';
  
  const props = defineProps({
    records: {
      type: Array,
      required: true,
    },
  });
  </script>
  