<template>
  <v-data-table :headers="headers" :items="desserts" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>CURD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              Booking
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-form ref="form" v-model="valid" lazy-validation>
                  <v-row>
                    <input v-model="editedItem.userId" type="hidden" />
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field outlined v-model="editedItem.firstName" :rules="firstNameRules" label="First name">
                      </v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field outlined v-model="editedItem.lastName" :rules="lastNameRules" label="Last name">
                      </v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field outlined v-model="editedItem.phone" :rules="phoneRules" label="Phone"></v-text-field>
                    </v-col>
                  </v-row>
                </v-form>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">
                Cancel
              </v-btn>
              <v-btn :disabled="!valid" color="blue darken-1" text @click="save">
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogResponse" width="500">
          <v-card>
            <v-card-title class="text-h5" :class="dialogColor">
              <span v-text="dialogTitle"></span>
            </v-card-title>

            <v-card-text class="pt-5">
              <span v-text="dialogMessage"></span>
            </v-card-text>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" text @click="dialogResponse = false">
                I accept
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:[`item.actions`]="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon small @click="deleteItem(item)">
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: 'First name', value: 'firstName' },
      { text: 'Last name', value: 'lastName' },
      { text: 'Phone', value: 'phone' },
      { text: 'Actions', value: 'actions', sortable: false },
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      userId: '',
      firstName: '',
      lastName: '',
      phone: '',
    },
    defaultItem: {
      userId: '',
      firstName: '',
      lastName: '',
      phone: '',
    },
    valid: true,
    firstNameRules: [
      v => !!v || 'First name is required',
    ],
    lastNameRules: [
      v => !!v || 'Last name is required',
    ],
    phoneRules: [
      v => !!v || 'Phone is required',
    ],
    dialogResponse: false,
    dialogTitle: '',
    dialogMessage: '',
    dialogColor: '',
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Booking' : 'Edit Booking'
    },
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },

  created() {
    this.initialize()
  },

  methods: {
    initialize() {
      this.userGet()
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      console.log(item)
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    deleteItemConfirm() {
      this.userDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      if (this.$refs.form.validate()) {
        if (this.editedIndex > -1) {
          this.userEdit()
        } else {
          this.userCreate()
        }
        this.close()
      }
    },

    async userCreate() {
      const temStatus = {
        userId: '',
        firstName: this.editedItem.firstName,
        lastName: this.editedItem.lastName,
        phone: this.editedItem.phone,
      }
      const formData = new FormData()
      formData.append('first_name', this.editedItem.firstName)
      formData.append('last_name', this.editedItem.lastName)
      formData.append('phone', this.editedItem.phone)
      try {
        const respones = await this.$axios.post('v1/user/create', formData)
        if (respones.data.code === 200 && respones.data.status === 'success') {
          this.dialogColor = 'success'
          this.dialogTitle = 'Success'
          this.dialogMessage = 'Create Success!'
          this.dialogResponse = true
          temStatus.userId = respones.data.respones.id
          this.desserts.unshift(temStatus)
        }
      } catch (err) {
        this.dialogColor = 'error'
        this.dialogTitle = 'Oops...'
        this.dialogMessage = err.response.data.message
        this.dialogResponse = true
      }
    },

    async userEdit() {
      const temIndex = this.editedIndex
      const temStatus = {
        userId: this.editedItem.userId,
        firstName: this.editedItem.firstName,
        lastName: this.editedItem.lastName,
        phone: this.editedItem.phone,
      }
      const formData = new FormData()
      formData.append('first_name', this.editedItem.firstName)
      formData.append('last_name', this.editedItem.lastName)
      formData.append('phone', this.editedItem.phone)
      try {
        const respones = await this.$axios.post(`v1/user/${this.editedItem.userId}/update`, formData)
        if (respones.data.code === 200 && respones.data.status === 'success') {
          this.dialogColor = 'success'
          this.dialogTitle = 'Success'
          this.dialogMessage = 'Edit Success!'
          this.dialogResponse = true
          Object.assign(this.desserts[temIndex], temStatus)
        }
      } catch (err) {
        this.dialogColor = 'error'
        this.dialogTitle = 'Oops...'
        this.dialogMessage = err.response.data.message
        this.dialogResponse = true
      }
    },

    async userGet() {
      try {
        const respones = await this.$axios.get('v1/user')
        if (respones.data.code === 200 && respones.data.status === 'success') {
          const rsArray = []
          respones.data.response.map(function (key, index) {
            return rsArray.push({
              'userId': respones.data.response[index].id,
              'firstName': respones.data.response[index].first_name,
              'lastName': respones.data.response[index].last_name,
              'phone': respones.data.response[index].phone,
            })
          });
          this.desserts = rsArray
        }
      } catch (err) {
        console.log(err)
      }
    },

    async userDelete() {
      const temIndex = this.editedIndex
      try {
        const respones = await this.$axios.post(`v1/user/${this.editedItem.userId}/delete`)
        if (respones.data.code === 200 && respones.data.status === 'success') {
          this.dialogColor = 'success'
          this.dialogTitle = 'Success'
          this.dialogMessage = 'Delete Success!'
          this.dialogResponse = true
          this.desserts.splice(temIndex, 1)
          this.closeDelete()
        }
      } catch (err) {
        this.dialogColor = 'error'
        this.dialogTitle = 'Oops...'
        this.dialogMessage = err.response.data.message
        this.dialogResponse = true
      }
    },
  },
}
</script>
