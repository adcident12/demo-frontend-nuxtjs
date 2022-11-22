<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card class="mx-auto pa-5 text-center" max-width="450" outlined>
        <div class="d-flex justify-center">
          <v-card-title>
            Login
          </v-card-title>
        </div>
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-text-field outlined v-model="phone" :rules="phoneRules" label="Phone" required>
          </v-text-field>

          <v-btn :disabled="!valid" color="success" class="mr-4" @click="validate">
            Login
          </v-btn>

          <div class="d-flex justify-center">
            <v-card-subtitle>
              Already have not an account? <NuxtLink class="text-decoration-none" to="/register">Register</NuxtLink>
            </v-card-subtitle>
          </div>
        </v-form>
      </v-card>
    </v-col>
    <v-dialog v-model="dialog" width="500">
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
          <v-btn color="primary" text @click="dialog = false">
            I accept
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  layout: 'clean',
  name: 'LoginPage',
  data: () => ({
    valid: true,
    phone: '',
    phoneRules: [
      v => !!v || 'Phone is required',
    ],
    dialog: false,
    dialogTitle: '',
    dialogMessage: '',
    dialogColor: '',
  }),

  methods: {
    validate() {
      if (this.$refs.form.validate()) {
        this.userLogin()
      }
    },
    async userLogin() {
      const formData = new FormData()
      formData.append('phone', this.phone)
      try {
        await this.$auth.loginWith('local', { data: formData })
      } catch (err) {
        this.dialogColor = 'error'
        this.dialogTitle = 'Oops...'
        this.dialogMessage = err.response.data.message
        this.dialog = true
      }
    }
  },
}
</script>
