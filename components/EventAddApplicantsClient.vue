<template>
  <div style="width: 100%;">
    <v-card>
      <v-divider></v-divider>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          color="grey"
          class="white--text"
          :to="`/events/${$route.params.eventId}/edit`"
        >
          Ubah
        </v-btn>
      </v-card-actions>
    </v-card>
    <v-row>
      <v-col cols="auto">
        <v-text-field
          v-model="filterSearch"
          label="Nama Kegiatan"
          clearable
          outlined
          dense
        />
      </v-col>
      <v-spacer></v-spacer>
      <v-col cols="auto">
        <v-btn color="primary" to="/events/create">
          Tambah Kegiatan
        </v-btn>
      </v-col>
    </v-row>
    <v-card>
      <v-data-table
        :headers="headers"
        :items="records"
        :loading="loading"
        fixed-header
      >
        <template v-slot:item.start_at="{ value }">
          {{ $dateFns.format(new Date(value), 'dd MMMM yyyy HH:mm') }}
        </template>
        <template v-slot:item.end_at="{ value }">
          {{ $dateFns.format(new Date(value), 'dd MMMM yyyy HH:mm') }}
        </template>
        <template v-slot:item.status="{ value }">
          <v-chip small class="ma-2" :color="value | getChipColor">
            {{ value }}
          </v-chip>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon
            v-if="allow.includes('view-events')"
            class="mr-2"
            @click="$router.push(`events/${item.id}`)"
          >
            mdi-card-search
          </v-icon>
          <v-icon
            v-if="allow.includes('manage-events')"
            class="mr-2"
            @click="$router.push(`events/${item.id}/edit`)"
          >
            mdi-pencil
          </v-icon>
          <v-icon
            v-if="allow.includes('manage-events')"
            @click="remove(item.id)"
          >
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script>
import { getChipColor } from '@/utilities/formater'
const headers = [
  { text: 'Nama Kegiatan', value: 'event_name', sortable: false, width: 150 },
  { text: 'Tanggal Mulai', value: 'start_at', width: 180 },
  { text: 'Tanggal Selesai', value: 'end_at', width: 180 },
  { text: 'Lokasi', value: 'event_location', width: 200 },
  { text: 'Kota/Kab', value: 'city.name', width: 200 },
  { text: 'Peserta', value: 'invitations_count', width: 100 },
  { text: 'Kloter', value: 'schedules_count', width: 100 },
  { text: 'Status', value: 'status', width: 200 },
  { text: 'Actions', value: 'actions', sortable: false, width: 150 }
]

export default {
  filters: {
    getChipColor
  },

  props: {
    title: {
      type: String,
      default: 'Kegiatan'
    },

    allow: {
      type: Array,
      default: () => []
    }
  },

  data() {
    return {
      headers,
      filterSearch: null
    }
  },

  computed: {
    records() {
      return this.$store.getters['events/getList']
    },
    loading() {
      return this.$store.getters['events/getLoading']
    },
    options: {
      set(value) {
        this.$store.commit('events/SET_TABLE_OPTIONS', value)
      },
      get() {
        return this.$store.getters['events/getTableOption']
      }
    }
  },

  watch: {
    options(value) {
      this.$emit('optionChanged', value)
    }
  },

  mounted() {
    this.options.perPage = 5000
    this.getRecords()
  },

  methods: {
    getRecords() {
      this.$store.dispatch('events/getList')
    },
    remove(id) {
      try {
        this.$store.dispatch('events/remove', id)
      } catch (error) {}
    }
  }
}
</script>
