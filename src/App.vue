<template>
 <div class="name">{{ greeting(name) }}</div>
 <div>{{ greet }}</div>
 <div>{{ name }}</div>
 <button 
 class="btn btn-primary"
 @click="updateName()"
 >Click</button>
</template>

<script setup>
import{ref} from 'vue';

let name = ref('Nayoung');

const greeting = (name) =>{
return 'hello,'+name;
};

const greet = greeting(name.value);

const updateName = () =>{
  name.value = 'Park Nayoung';
  console.log(name);
};
</script>

<style>
.name{
  color : pink;
}
</style>
