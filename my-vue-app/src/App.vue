<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6">
        <FormComponent @add-record="addRecord" />
      </div>
      <div class="col-md-6">
        <TableComponent :records="records" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';
import 'bootstrap/dist/css/bootstrap.css'
import FormComponent from './components/FormComponent.vue';
import TableComponent from './components/TableComponent.vue';

const records = ref([]);

const addRecord = async () => {
  const options = {
    method: 'GET',
    url: 'https://eh0164rwce.execute-api.us-east-1.amazonaws.com/default/taller-lamda',
    params: {TableName: 'taller-formulario'},
    headers: {'content-type': 'application/json'}
  };

  try {
    const { data } = await axios.request(options);
    console.log(data);
    records.value = data.Items;
  } catch (error) {
    console.error(error);
  }
};

onMounted(async ()=>{
  await addRecord()
})
</script>

<style>
.container {
  margin-top: 20px;
}
</style>
