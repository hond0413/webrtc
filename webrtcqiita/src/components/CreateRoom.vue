<template>
  <v-card>
    <v-card-text>
      <v-text-field v-model="roomName" label="RoomName" required />
    </v-card-text>
    <v-card-actions>
      <v-btn @click="roomCreate(roomName)">
        Create
      </v-btn>
      <v-btn @click="clear()">
        Clear
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
export default {
  data() {
    return {
      roomName: ""
    }
  },
  methods: {
    roomCreate(val) {
      this.$router.push({path: 'video', query: { name: val }})
    },
    clear() {
      this.roomName = ""
    }
  }
}
</script>
