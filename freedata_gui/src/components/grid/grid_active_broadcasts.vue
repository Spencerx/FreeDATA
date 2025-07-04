<script setup>
import { ref } from "vue";
import { setActivePinia } from "pinia";
import pinia from "../../store/index";
import { useStateStore } from "../../store/stateStore.js";
import { sendModemCQ, sendModemPing, setModemBeacon } from "../../js/api.js";

// Set the active Pinia store
setActivePinia(pinia);
const state = useStateStore(pinia);

// Reactive reference for DX Call
const dxcallPing = ref("");

// Reactive references to manage button states
const isCQButtonDisabled = ref(false);
const isPingButtonDisabled = ref(false);



// Function to transmit a ping
async function transmitPing() {
  isPingButtonDisabled.value = true;

  // Send Ping message
  await sendModemPing(dxcallPing.value.toUpperCase());

  // Wait for 6 seconds (cooldown period)
  setTimeout(() => {
    isPingButtonDisabled.value = false;
  }, 6000);
}

// Function to send CQ and handle button disable and cooldown
async function handleSendCQ() {
  isCQButtonDisabled.value = true;

  // Send CQ message
  await sendModemCQ();

  // Wait for 6 seconds (cooldown period)
  setTimeout(() => {
    isCQButtonDisabled.value = false;
  }, 10000);
}


// Function to start or stop the beacon
function startStopBeacon() {
  if (state.beacon_state) {
    setModemBeacon(false);
  } else {
    setModemBeacon(true);
  }
}

// Listen for the stationSelected event and update dxcallPing
window.addEventListener(
  "stationSelected",
  function (eventdata) {
    const evt = eventdata;
    dxcallPing.value = evt.detail;
  },
  false
);
</script>

<template>
  <div class="card h-100">
    <div class="card-header">
      <i
        class="bi bi-broadcast"
        style="font-size: 1.2rem"
      />&nbsp;
      <strong>{{ $t('grid.components.broadcasts') }}</strong>
    </div>
    <div class="card-body overflow-auto">
      <div class="input-group">
        <input
          v-model="dxcallPing"
          type="text"
          class="form-control"
          style="max-width: 6rem; min-width: 3rem; text-transform: uppercase"
          placeholder="DXcall"
          pattern="[A-Z]*"
          maxlength="11"
          aria-label="Input group"
          aria-describedby="btnGroupAddon"
        >
        <button
          id="sendPing"
          class="btn btn-outline-secondary"
          type="button"
          data-bs-placement="bottom"
          data-bs-toggle="tooltip"
          data-bs-trigger="hover"
          data-bs-html="false"
          title="Send a ping request to a remote station"
          @click="transmitPing"
        >
          <strong v-if="!isPingButtonDisabled">{{ $t('grid.components.pingstation') }}</strong>
          <strong v-else>{{ $t('grid.components.transmitting') }}</strong>
        </button>

        <button
          id="sendCQ"
          class="btn btn-outline-secondary ms-1"
          type="button"
          :title=" $t('grid.components.cqcqcq_help')"
          @click="handleSendCQ"
        >
          <span v-if="!isCQButtonDisabled">{{ $t('grid.components.callcq') }}</span>
          <span v-else>{{ $t('grid.components.sendingcq') }}</span>
        </button>

        <button
          id="startBeacon"
          type="button"
          class="btn ms-1"
          :class="{
            'btn-success': state.beacon_state,
            'btn-outline-secondary': !state.beacon_state,
          }"
          :title=" $t('grid.components.togglebeacon_help')"
          @click="startStopBeacon"
        >
          {{ $t('grid.components.togglebeacon') }}
        </button>
      </div>
      <!-- end of row-->
    </div>
  </div>
</template>
