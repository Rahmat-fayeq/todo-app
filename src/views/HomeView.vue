<script setup lang="ts">
  import { ref, watch,computed, type Ref } from 'vue';
  import {uid} from 'uid';
  import TodoForm from '../components/TodoForm.vue' 
  import TodoItem from '../components/TodoItem.vue' 
  import draggable from 'vuedraggable'
  import {Icon} from "@iconify/vue"

  type TodoType = {
    id: string,
    todo: string,
    isComplete: boolean,
    isEditing: boolean,
  }

  const todoList:Ref<TodoType[]> = ref([]);

  watch(todoList, () => {
    setTodoListToLocalStorage();
  },{deep: true});

  const todoComplate = computed(()=>{
    return todoList.value.every((todo)=>todo.isComplete);
  });

  const fetchTodoListFromLocalStorage =()=>{
    const savedTodo = JSON.parse(localStorage.getItem("todoList") as any);
    if(savedTodo){
      todoList.value = savedTodo
    }
  }
  fetchTodoListFromLocalStorage();

  const setTodoListToLocalStorage = () =>{
    localStorage.setItem("todoList", JSON.stringify(todoList.value));
  }


  const createTodo = (todo:string) =>{

    todoList.value.push({
      id: uid(),
      todo,
      isComplete: false,
      isEditing:false,
    });
  }
  const handleToggle = (index:number) =>{
    todoList.value[index].isComplete = !todoList.value[index].isComplete
  }

  const handleDelete =(id:string)=>{
    todoList.value = todoList.value.filter((todo)=>todo.id !== id);
  }

  const handleEdit=(index:number)=>{
    todoList.value[index].isEditing = !todoList.value[index].isEditing;
  }

  const handleUpdate = (todo:string, index:number) =>{
    todoList.value[index].todo = todo
  }

</script>

<template>
  <main>
    <h1>Create Todo</h1>
    <TodoForm  @create-todo="createTodo" />
    <draggable v-model="todoList" tag="ul" item-key="todo" class="todo-list" v-if="todoList.length>0">
      <template #item="{element:todo,index}">
         <TodoItem 
        :todo="todo" 
        :index="index" 
        :key="todo.id" 
        @toggle-todo="handleToggle" 
        @delete-todo="handleDelete"
        @edit-todo="handleEdit"
        @update-todo="handleUpdate"
      />
      </template>
    </draggable>
    <!-- <ul class="todo-list" v-if="todoList.length>0">
      <TodoItem 
        v-for="(todo,index) in todoList" 
        :todo="todo" 
        :index="index" 
        :key="todo.id" 
        @toggle-todo="handleToggle" 
        @delete-todo="handleDelete"
        @edit-todo="handleEdit"
        @update-todo="handleUpdate"
      />
    </ul> -->
    <p class="todos-msg" v-else>
      <Icon icon="noto-v1:sad-but-relieved-face" />
      <span>You have no todo's to complete! Add one!</span>
    </p>
    <p class="todos-msg" v-if="todoComplate && todoList.length>0">
      <Icon icon="noto-v1:party-popper" />
      <span>Bravo! You have complated all your todo.</span>
    </p>
  </main>
</template>

<style scoped lang="scss">
main {
  display: flex;
  flex-direction: column;
  max-width: 500px;
  width: 100%;
  margin: 0 auto;
  padding: 40px 16px;

  h1 {
    margin-bottom: 16px;
    text-align: center;
  }

  .todo-list {
    display: flex;
    flex-direction: column;
    list-style: none;
    margin-top: 24px;
    gap: 20px;
  }

  .todos-msg {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    margin-top: 24px;
  }
}
</style>

