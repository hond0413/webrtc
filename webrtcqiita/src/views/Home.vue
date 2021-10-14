<template>
  <div>
    <RoomTable />
    <CreateRoom />
  </div>
</template>

<script>
import Vue from 'vue'
import RoomTable from '../components/RoomTable'
import CreateRoom from '../components/CreateRoom'
export default Vue.extend({
    components: {
        RoomTable,
        CreateRoom
    }
})
</script>