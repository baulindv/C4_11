<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <div>
        <b-alert v-model="showDismissibleAlert" variant="success"

        dismissible>{{ confirmationMessage }}</b-alert>
      </div>
      <button type="button"
              id="task-add"
              class="btn btn-success btn-sm align-left d-block"
              v-b-modal.todo-modal>
        Добавить задачу
      </button>
      <div>
        <p >Всего задач: {{ all_todos }} &nbsp;
          Выполнено {{ done }}
        &nbsp; Не выполнено: {{ undone }}</p>
      </div>
      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                        class="btn btn-secondary btn-sm"
                        v-b-modal.todo-update-modal
                        @click="updateTodo(todo)"
                        >
                    Обновить
                </button>
                &nbsp;
                <button type="button"
                  class="btn btn-danger btn-sm"
                  @click="deleteTodo(todo)">
                    X
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <b-modal ref="addTodoModal"
      id="todo-modal"
      title="Добавить задачу"
      hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-description-group"
                  label="Описание:"
                  label-for="form-description-input">
          <b-form-input id="form-description-input"
                  type="text"
                  v-model="addTodoForm.description"
                  required
                  placeholder="Завести задачу">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-complete-group">
          <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Добавить</b-button>
        <b-button type="reset" variant="danger">Сброс</b-button>
      </b-form>
    </b-modal>
    <b-modal ref="updateTodoModal"
            id="todo-update-modal"
            title="Update"
            hide-footer>
      <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
        <b-form-group id="form-update-description-group"
                label="Описание:"
                label-for="form-update-description-input">
          <b-form-input id="form-update-description-input"
                  type="text"
                  v-model="updateTodoForm.description"
                  required>
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-update-complete-group">
          <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">Обновить</b-button>
          <b-button type="reset" variant="danger">Сброс</b-button>
        </b-button-group>
      </b-form>
    </b-modal>
  </div>
</template>
<style>
  button#task-add {
    margin-top: 20px;
    margin-bottom: 20px;
  }
  h1, td {
    text-align: left;
  }
  .todo-uid {
    text-align: right;
  }
</style>

<script>

export default {
  name: 'Todos',
  data() {
    return {
      todos: [],
      addTodoForm: {
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      // props: ['message'],
      confirmationMessage: '',
      showDismissibleAlert: false,
    };
  },

  methods: {
    getTodos() {
      // console.log(localStorage.getItem('todos'));
      this.todos = JSON.parse(localStorage.getItem('todos'));
      const itemQuantity = this.todos.length;
      this.all_todos = itemQuantity;
      const array = JSON.parse(localStorage.getItem('todos'));

      const isTrue = array.filter(item => item.is_completed === 'true');

      this.done = isTrue.length;
      this.undone = this.all_todos - this.done;
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      const requestData = {
        description: this.addTodoForm.description,
        is_completed: this.addTodoForm.is_completed[0],
      };

      let localStorageList = JSON.parse(localStorage.getItem('todos'));
      if (localStorageList === undefined || !Array.isArray(localStorageList)) {
        localStorageList = [];
      }

      localStorageList.push({
        uid: (new Date()).getTime(),
        description: this.addTodoForm.description,
        is_completed: this.addTodoForm.is_completed[0],

      });
      localStorage.setItem('todos', JSON.stringify(localStorageList));

      this.getTodos();
      this.confirmationMessage = `Задача "${requestData.description}" добавлена`;
      this.showDismissibleAlert = true;
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
    },
    updateTodo(todo) {
      this.updateTodoForm = todo;
    },
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      // const requestData = {
      //   description: this.updateTodoForm.description,
      //   is_completed: this.updateTodoForm.is_completed[0],
      // };

      // console.log(this.updateTodoForm.uid);

      const localStorageList = JSON.parse(localStorage.getItem('todos'));
      // console.log(localStorageList);
      const newList = [];
      for (const obj of localStorageList) {
        if (obj.uid === this.updateTodoForm.uid) {
          newList.push({
            uid: this.updateTodoForm.uid,
            description: this.updateTodoForm.description,
            is_completed: this.updateTodoForm.is_completed[0],
          });
        } else {
          newList.push(obj);
        }
      }
      localStorage.setItem('todos', JSON.stringify(newList));

      this.getTodos();
      this.showDismissibleAlert = true;
      this.confirmationMessage = 'Задача обновлена';
    },
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
    },
    deleteTodo(todo) {
      // console.log(todo.uid);
      const localStorageList = JSON.parse(localStorage.getItem('todos'));
      // console.log(localStorageList);
      const newList = [];
      for (const element of localStorageList) {
        if (element.uid !== todo.uid) {
          newList.push(element);
        }
      }
      localStorage.setItem('todos', JSON.stringify(newList));

      this.getTodos();
      this.showDismissibleAlert = true;
      this.confirmationMessage = 'Задача удалена из списка';
    },
  },
  created() {
    this.getTodos();
  },
};
</script>
