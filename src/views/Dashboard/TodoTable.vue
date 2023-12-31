<template>
  <div class="card">
    <div class="card-header pb-0">
      <h6>ToDo List table</h6>
    </div>
    <div class="card-body px-0 pt-0 pb-2">
      <div class="table-responsive p-0">
        <table class="table align-items-center mb-0">
          <thead>
            <tr>
              <th>No</th>
              <th>Nama</th>
              <th>Jobdesk</th>
              <th>Status</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in g$list" :key="item.id">
              <td class="ps-4">{{ index + 1 }}</td>
              <td>
                <h6>{{ item.title }}</h6>
              </td>
              <td>{{ item.description }}</td>
              <td class="text-primary ps-4">
                {{ item.completed ? "Done" : "Not Yet" }}
              </td>
              <td>
                <button
                  class="btn-secondary rounded px-3"
                  @click="showModal(item)"
                >
                  Edit
                </button>

                <button
                  class="btn-danger rounded px-3 ms-2"
                  @click="showConfirmation(item.id, item.title)"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <Modal v-show="confirmationVisible" @close="closeModal2">
        <template v-slot:header>This Modal Confirmation</template>
        <template v-slot:body>
          <div class="mt-3">
            <div class="text-center fs-1">
              Are you sure to delete {{ deleteItemTitle }}?
            </div>
            <div class="text-center mt-9">
              <button
                class="btn btn-danger px-6 fs-5"
                @click="deleteItem"
                type="submit"
              >
                Delete
              </button>
              <button
                class="btn btn-secondary px-6 fs-5 ms-2"
                @click="closeModal2"
                type="submit"
              >
                Cancel
              </button>
            </div>
          </div>
        </template>
        <template v-slot:footer>@footerKonfirmasi</template>
      </Modal>

      <Modal v-show="isModalEditVisible" @close="closeModal">
        <template v-slot:header>This Modal</template>
        <template v-slot:body>
          <div class="mt-3">
            <form v-on:submit.prevent="updateList">
              <div class="form-group">
                <label for="title">Nama:</label>
                <input
                  type="text"
                  class="form-control"
                  id="title"
                  v-model="editedItem.title"
                />
              </div>
              <div class="form-group">
                <label for="completed">Status</label>
                <argon-switch
                  @change="toggleSwitch"
                  v-model="editedItem.completed"
                >
                  {{ editedItem.completed ? "Done" : "Not Yet" }}
                </argon-switch>
              </div>
              <div class="form-group">
                <label for="description">JobDesk:</label>
                <textarea
                  type="text"
                  class="form-control"
                  id="description"
                  v-model="editedItem.description"
                ></textarea>
              </div>
              <div class="text-center mt-4">
                <button class="btn btn-primary px-6 fs-5" type="submit">
                  Save
                </button>
              </div>
            </form>
          </div>
        </template>
        <template v-slot:footer>@footerMeme</template>
      </Modal>
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from "pinia";
import Modal from "@/components/Modal.vue";
import ArgonSwitch from "@/components/ArgonSwitch.vue";
import d$todo from "@/stores/dashboard/todo";

export default {
  name: "ToDoTable",
  components: {
    ArgonSwitch,
    Modal,
  },
  data() {
    return {
      editedItem: {
        id: "",
        title: "",
        description: "",
        completed: false,
      },
      isModalEditVisible: false,
      confirmationVisible: false,
      deleteItemId: null,
      deleteItemTitle: "",
    };
  },
  computed: {
    ...mapState(d$todo, ["g$list"]),
  },
  methods: {
    showModal(item) {
      this.editedItem = { ...item };
      this.isModalEditVisible = true;
    },
    showConfirmation(itemId, title) {
      this.confirmationVisible = true;
      this.deleteItemId = itemId;
      this.deleteItemTitle = title;
    },
    closeModal() {
      this.isModalEditVisible = false;
    },
    closeModal2() {
      this.confirmationVisible = false;
    },
    async updateList() {
      try {
        const updatedItem = { ...this.editedItem };
        delete updatedItem.id;

        // Periksa apakah deskripsi tidak kosong
        if (!updatedItem.description) {
          throw new Error("Description is required.");
        }

        await this.a$update(this.editedItem.id, updatedItem);

        // Perbarui daftar tugas secara lokal dengan tugas yang diperbarui
        const index = this.g$list.findIndex(
          (item) => item.id === this.editedItem.id
        );
        if (index !== -1) {
          this.g$list[index] = { ...this.g$list[index], ...updatedItem };
        }

        this.closeModal();

        // Perbarui data tabel secara otomatis setelah operasi pembaruan berhasil
        await this.getList();
      } catch (e) {
        console.error(e);
      }
    },

    async deleteItem() {
      try {
        await this.a$delete(this.deleteItemId);

        // Hapus item dari daftar tugas secara lokal
        this.g$list = this.g$list.filter(
          (item) => item.id !== this.deleteItemId
        );

        // Perbarui data tabel secara otomatis setelah operasi penghapusan berhasil
        await this.getList();

        this.closeModal2();
      } catch (e) {
        console.error(e);
      }
    },

    ...mapActions(d$todo, ["a$list", "a$update", "a$delete"]),
    async getList() {
      try {
        await this.a$list();
      } catch (e) {
        console.error("methods getList error", e);
      }
    },
    toggleSwitch() {
      this.editedItem.completed = !this.editedItem.completed;
    },
  },
  async created() {
    await this.getList();
  },
};
</script>
