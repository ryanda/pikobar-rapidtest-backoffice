<template>
  <v-card>
    <v-card-title>
      <p class="text-h6 text--primary">
        Tambah Kegiatan
      </p>
    </v-card-title>
    <v-divider></v-divider>
    <validation-observer
      v-slot="{ valid, handleSubmit }"
      tag="div"
      class="v-card__text"
    >
      <form class="row justify-end" @submit.prevent="handleSubmit(doStore)">
        <v-col v-if="type === 'edit'" cols="12" class="text-right">
          Status:
          <v-btn-toggle v-model="status" mandatory color="primary" class="pl-2">
            <v-btn small value="DRAFT">
              DRAFT
            </v-btn>
            <v-btn small value="PUBLISHED">
              PUBLISHED
            </v-btn>
          </v-btn-toggle>
        </v-col>
        <v-col cols="6">
          <pkbr-input
            v-model="event_name"
            name="Nama Kegiatan"
            label="Nama Kegiatan"
            placeholder="Masukan Nama Kegiatan"
            rules="required"
          />
        </v-col>
        <v-col cols="6">
          <pkbr-input
            v-model="host_name"
            name="Penyelenggara"
            label="Penyelenggara"
            placeholder="Masukan Instansi Penyelenggara"
            rules="required"
          />
        </v-col>
        <v-col cols="6">
          <pkbr-input
            v-model="event_location"
            name="Lokasi"
            label="Lokasi"
            placeholder="Masukan Alamat Lokasi"
            rules="required"
          />
        </v-col>
        <v-col cols="6">
          <pkbr-select
            v-model="city_code"
            :items="getKabkot"
            label="Kab./Kota"
            name="Kab./Kota"
            placeholder="Pilih Kab./Kota"
            rules="required"
            item-text="name"
            item-value="code"
          />
        </v-col>
        <v-col cols="6">
          <pkbr-input-date
            v-model="tanggal"
            label="Tanggal"
            name="Tanggal"
            placeholder="Masukan Tanggal Kegiatan"
            rules="required"
          />
        </v-col>
        <v-col cols="6">
          <pkbr-input-time
            v-for="(waktu, i) in kloter"
            :key="i"
            v-model="kloter[i]"
            :label="`Kloter ${i + 1}`"
            :name="`Kloter ${i + 1}`"
            placeholder="Masukan Jam Kloter"
            rules="required|time_range"
          >
            <template v-if="type === 'create'" v-slot:append-outer>
              <v-btn small icon color="success" @click="addKloter">
                <v-icon>mdi-plus</v-icon>
              </v-btn>
              <v-btn
                :disabled="i < 1"
                small
                icon
                color="error"
                @click="removeKloter(i)"
              >
                <v-icon v-if="i > 0">mdi-minus</v-icon>
              </v-btn>
            </template>
          </pkbr-input-time>
        </v-col>
        <v-col cols="auto">
          <v-btn color="success" :disabled="!valid" type="submit">
            Simpan
          </v-btn>
        </v-col>
      </form>
    </validation-observer>
  </v-card>
</template>

<script>
/* eslint-disable camelcase */
import { mapGetters } from 'vuex'
import { format } from 'date-fns'

export default {
  props: {
    formData: {
      type: Object,
      default: null
    },
    type: {
      type: String,
      default: 'create'
    }
  },

  data() {
    return {
      event_name: null,
      host_name: null,
      event_location: null,
      city_code: null,
      status: 'DRAFT',
      tanggal: null,
      kloter: [null]
    }
  },

  computed: {
    ...mapGetters('area', ['getKabkot'])
  },

  watch: {
    formData(val) {
      let kloter = [null]
      if (val) {
        kloter = val.schedules.map((sch) => {
          const scheduleStart = new Date(sch.start_at)
          const scheduleEnd = new Date(sch.end_at)

          const inputScheduleStart = format(scheduleStart, 'HH:mm')
          const inputScheduleEnd = format(scheduleEnd, 'HH:mm')

          return `${inputScheduleStart}-${inputScheduleEnd}`
        })
      }
      this.event_name = val ? val.event_name : null
      this.status = val ? val.status : null
      this.host_name = val ? val.host_name : null
      this.event_location = val ? val.event_location : null
      this.city_code = val && val.city ? val.city.code : null
      this.tanggal = val
        ? this.$dateFns.format(
            val.start_at.split(':')[0],
            "yyyy-MM-dd'T'HH:mm:ssxxx"
          )
        : null
      this.kloter = kloter
    }
  },

  methods: {
    addKloter() {
      this.kloter.push(null)
    },

    removeKloter(index) {
      this.kloter.splice(index, 1)
    },

    doStore() {
      const schedules = this.kloter.map((waktu, i) => {
        let [start_atSch, end_atSch] = waktu.split('-')
        const { setHours, setMinutes } = this.$dateFns

        start_atSch = setHours(
          setMinutes(new Date(this.tanggal), start_atSch.split(':')[1]),
          start_atSch.split(':')[0]
        )

        end_atSch = setHours(
          setMinutes(new Date(this.tanggal), end_atSch.split(':')[1]),
          end_atSch.split(':')[0]
        )

        const id = (this.formData && this.formData.schedules[i].id) || null
        return id
          ? {
              id,
              start_at: start_atSch,
              end_at: end_atSch
            }
          : {
              start_at: start_atSch,
              end_at: end_atSch
            }
      })
      const start_at = schedules[0].start_at
      const end_at = schedules[schedules.length - 1].end_at
      const data = {
        event_name: this.event_name,
        host_name: this.host_name,
        event_location: this.event_location,
        city_code: this.city_code,
        status: this.status,
        start_at,
        end_at,
        schedules
      }
      this.$emit('onStore', data)
    }
  }
}
</script>
