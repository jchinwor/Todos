<template>
 <div class="todo_contain">

    <div v-if="showToast">
         <fwb-toast closable type="success">
            Todo deleted successfully.
        </fwb-toast>
    </div>

    <h2 class="text-center mb-4">
        Todo APP
    </h2>

      <div class="mb-5">
          <fwb-input
            v-model="todoInput"
            label=""
            placeholder="Add to do"
            size="lg"
            
        >
            <template #prefix>
            
            </template>
            <template #suffix>
            <fwb-button color="green" :disabled="!todoInput" @click="addTodo">Add</fwb-button>
            </template>
        </fwb-input>
      </div>
        
      <fwb-card href="#"  v-for="todo in todos" :key="todo" class="mb-4 bg-green-100" >
        <p class="p-3 text-center">
            {{ new Date(todo.date).toLocaleString("en-us", { dateStyle: "long"}) }}
        </p>
      <div class="p-5 flex justify-between justify-items-center items-center break-all" :class="{'bg-green-100' : todo.done}">
        
        <div class="">
            <p class="font-normal text-gray-700 dark:text-gray-400" :class="{'text-green-800 line-through' : todo.done}">
            {{todo.content}}
            </p>
        </div>
       <div class="flex ">

         <div v-if="todo.done" class="mr-3">
             <fwb-button color="green" @click="toggleDone(todo.id)">&check;</fwb-button>
         </div>
         <div v-else>
            <fwb-button color="light" @click="toggleDone(todo.id)" class="mr-3">&check;</fwb-button>
         </div>

         <!-- <fwb-button  class="mr-3" :class="todo.done ? 'bg-green-600' : 'bg-neutral-300'">&check;</fwb-button>
         -->
         <fwb-button color="red" @click="deleteTodo(todo.id)">&cross;</fwb-button>
       </div>
      </div>
    </fwb-card>

 </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import { FwbButton, FwbInput } from 'flowbite-vue'
import { FwbCard } from 'flowbite-vue'
import { db } from '../firebase/index'
import { collection, doc, deleteDoc , setDoc, onSnapshot, updateDoc,query, orderBy, limit } from 'firebase/firestore';
import { FwbToast } from 'flowbite-vue'

const todosCollectionQuery = query(collection(db, 'Todos'), orderBy("date", "desc"), limit(10));

const todos = ref([
]) 

const showToast = ref(false)
const todoInput = ref('')

onMounted(() =>{


  onSnapshot(todosCollectionQuery, (querySnapshot) => {
     let fbTodos = []
  querySnapshot.forEach((doc) => {
        const todo = {
            id: doc.id,
            content: doc.data().content,
            done:doc.data().done,
            date:doc.data().date
        } 

        fbTodos.push(todo)
  });

   todos.value = fbTodos
 
});


})

const addTodo = () =>{
        // const dataBase = await db.collection("Todos").doc();
        const dataBase = doc(collection(db, "Todos"));
        const timestamp = Date.now();

         setDoc(dataBase, { 
            id: dataBase.id,
            content: todoInput.value,
            done: false,
            date: timestamp,
        });
    // todos.value.unshift({
    //     content:todoInput.value,
    //     done:false
    // })
    todoInput.value = ''
} 

const deleteTodo = (id) =>{

    // todos.value = todos.value.filter((todo)=>todo.id !== id)
     deleteDoc(doc(db, "Todos", id));
     showToast.value = true

}

const toggleDone = (id) =>{

    const databaseCollectionRef = doc(db, "Todos", id);
    const index = todos.value.findIndex(todo => todo.id === id)

    // Set the "capital" field of the city 'DC'
     updateDoc(databaseCollectionRef, {
      done: !todos.value[index].done
    });

    // const index = todos.value.findIndex(todo => todo.id === id)
    // todos.value[index].done = !todos.value[index].done
}

</script>

<style >
.todo_contain{
 max-width: 400px;
 padding:20px;
 margin:0 auto;
}
</style>
