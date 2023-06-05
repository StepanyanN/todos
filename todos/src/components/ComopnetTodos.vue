<template>
  <v-app>
    <!-- Input field -->
    <div class="inpDiv">
      <input class="inp" type="number" v-model="userId" placeholder="User ID" />
      <input
        class="inp"
        type="number"
        v-model="quantity"
        placeholder="Quantity"
      />
      <v-btn @click="getData">Search</v-btn>
    </div>

    <!-- All data table items -->
    <v-data-table
      :headers="headers"
      :items="items"
      :items-per-page="-1"
      :single-expand="singleExpand"
      :expanded.sync="expanded"
      show-expand
      class="elevation-1"
    >
      <!-- New item -->
      <template v-slot:top>
        <v-toolbar flat>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="green" dark class="mb-2" v-bind="attrs" v-on="on">
                New Item
              </v-btn>
            </template>

            <!-- Creating new todos -->
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="newTodo.userId"
                        label="User Id"
                        type="number"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="newTodo.id"
                        label="Id"
                        type="number"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="newTodo.username"
                        label="Username"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="newTodo.email"
                        label="Email"
                        type="email"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="newTodo.title"
                        label="Title"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
                <v-btn color="blue darken-1" text @click="createTodo">
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>

      <!-- Info about table row -->
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">{{ item }}</td>
      </template>

      <!-- Delete and Edit buttons -->
      <template #[`item.actions`]="{ item }">
        <v-card-actions>
          <v-btn color="dark" dark big @click="editItem(item)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
          <v-btn color="red" small @click="deleteItem(item)">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </v-card-actions>
      </template>
    </v-data-table>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  // All data
  data() {
    return {
      editableTodo: null,
      editTodos: null,
      dialog: false,
      editDialog: null,
      dialogDelete: null,
      userId: null,
      id: null,
      username: null,
      email: null,
      title: null,
      quantity: null,
      items: [],
      expanded: [],
      singleExpand: false,
      headers: [
        { text: "User ID", align: "left", sortable: true, value: "userId" },
        { text: "ID", align: "left", sortable: true, value: "id" },
        { text: "User Name", align: "left", sortable: true, value: "username" },
        { text: "Email", align: "left", sortable: true, value: "email" },
        { text: "Title", align: "left", sortable: true, value: "title" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      editedIndex: null,
      editedItem: {},
      defaultItem: {
        userId: Number,
        id: Number,
        username: "",
        email: "",
        title: "",
      },
      newTodo: {
        userId: null,
        id: null,
        username: "",
        email: "",
        title: "",
      },
    };
  },

  // Rendering the data to Local
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.getData();
  },

  // Calling function "getData()"
  mounted() {
    this.getData();
  },

  methods: {
    // Get the result of APIs
    async getData() {
      try {
        const [todosResponse, usersResponse] = await Promise.all([
          axios.get("https://jsonplaceholder.typicode.com/todos"),
          axios.get("https://jsonplaceholder.typicode.com/users"),
        ]);

        const todos = todosResponse.data;
        const users = usersResponse.data;

        let filteredTodos = todos;
        if (this.userId !== null) {
          filteredTodos = filteredTodos.filter(
            (todo) => todo.userId === parseInt(this.userId)
          );
        }
        if (this.quantity !== null) {
          const quantity = parseInt(this.quantity);
          filteredTodos = filteredTodos.slice(0, quantity);
        }
        if (filteredTodos.length < this.quantity) {
          alert(filteredTodos.length);
          filteredTodos = todos;
          this.quantity = null;
          this.userId = null;
        }


        const mappedData = filteredTodos.map((todo) => {
          const user = users.find((user) => user.id === todo.userId);
          return {
            userId: todo.userId,
            title: todo.title,
            username: user.username,
            id: todo.id,
            email: user.email,
          };
        });

        this.items = mappedData;
      } catch (error) {
        console.log(error);
      }

      // this.quantityToZero();
    },

    // Function "Create Todos"
    createTodo() {
      if (this.editedItemIndex !== -1) {
        // Если редактируется существующий элемент, заменяем его в массиве
        this.$set(this.items, this.editedItemIndex, { ...this.newTodo });
        this.editedItemIndex = -1;
      } else {
        // Если создается новый элемент, добавляем его в конец массива
        this.items.push({ ...this.newTodo });
      }
      this.close();
    },

    // Function "Edit items"
    editItem(item) {
      this.editedItemIndex = this.items.indexOf(item);
      this.dialog = true;
      this.newTodo = { ...item };
    },

    // Function "Delete items"
    deleteItem(item) {
      const index = this.items.indexOf(item);
      if (index !== -1) {
        this.items.splice(index, 1);
      }
    },

    // Function "Close dialog"
    close() {
      this.dialog = false;
      this.clear();
    },

    // Function "Clear data"
    clear() {
      this.newTodo = {
        userId: null,
        id: null,
        username: "",
        email: "",
        title: "",
      };
    },

    // Function "Save data"
    save() {
      this.items.push(this.editedItem);
      this.close();
      this.clear();
    },
  },
};
</script>
<style>
.inpDiv {
  display: flex;
  justify-content: center;
  margin: 20px;
}

.inp {
  margin-right: 10px;
  border: 0.5px solid black;
  outline: none;
  font-size: 18px;
}
</style>