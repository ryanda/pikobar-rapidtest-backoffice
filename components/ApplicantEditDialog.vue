<template>
  <div>
    <v-dialog :value="open" persistent max-width="800">
      <v-card>
        <v-card-title>
          <span class="headline">Edit Peserta</span>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="registrationCode"
                  label="Nomor Pendaftaran"
                  filled
                  readonly
                />
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="name" label="Nama Peserta" outlined />
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="nik" label="NIK" outlined />
              </v-col>
              <v-col cols="12">
                <v-select
                  v-model="gender"
                  :items="[
                    { text: 'Laki-Laki', value: 'M' },
                    { text: 'Perempuan', value: 'F' }
                  ]"
                  label="Jenis Kelamin"
                  outlined
                ></v-select>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="phone_number"
                  label="Nomor Telepon"
                  outlined
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="city_code"
                  label="Kab/Kota Tinggal Sekarang"
                  outlined
                />
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="address"
                  label="Alamat Tempat Tinggal Sekarang"
                  outlined
                />
              </v-col>
              <v-col cols="12">
                <v-select
                  v-model="status"
                  :items="[
                    { text: 'Calon Peserta', value: 'NEW' },
                    { text: 'Disetujui', value: 'APPROVED' }
                  ]"
                  label="Status"
                  outlined
                ></v-select>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer />
          <v-btn color="blue darken-1" text @click="close">
            Batal
          </v-btn>
          <v-btn color="blue darken-1" text @click="save">
            Simpan
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  props: {
    open: {
      type: Boolean,
      default: false
    },

    recordId: {
      type: Number,
      default: null
    }
  },

  data() {
    return {
      registrationCode: null,
      name: null,
      nik: null,
      gender: null,
      city_code: null,
      address: null,
      phone_number: null,
      status: null
    }
  },

  watch: {
    recordId(value) {
      if (value) {
        this.fetchRecord()
      }
    }
  },

  methods: {
    async fetchRecord() {
      const id = this.recordId
      const { data } = await this.$axios.$get(`/rdt/applicants/${id}`)

      this.registrationCode = data.registration_code
      this.name = data.name
      this.nik = data.nik
      this.gender = data.gender
      this.city_code = data.city_code
      this.address = data.address
      this.phone_number = data.phone_number
      this.status = data.status
    },

    async save() {
      const id = this.recordId

      try {
        await this.$axios.$put(`/rdt/applicants/${id}`, {
          name: this.name,
          gender: this.gender,
          nik: this.nik,
          city_code: this.city_code,
          address: this.address,
          phone_number: this.phone_number,
          status: this.status
        })

        this.$emit('save')
      } catch (e) {
        //
      }
    },

    close() {
      this.$emit('close')
    }
  }
}
</script>
