<template>
    <div>
        <!-- HTML HERE -->
        <h2>New Template</h2>
    </div>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref, reactive, watch, onMounted, onBeforeUnmount, computed } from "vue";
import { useRoute, useRouter } from "vue-router";

import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from "pinia";



// VARIABLES
const router = useRouter();
const route = useRoute();

const Mqtt = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);


// FUNCTIONS
onMounted(() => {
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect(); // Connect to Broker located on the backend
    setTimeout(() => {
        // Subscribe to each topic
        Mqtt.subscribe("620142646");
        Mqtt.subscribe("620142646_pub");
    }, 3000);
});



onBeforeUnmount(() => {
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    // unsubscribe from all topics
    Mqtt.unsubcribeAll();
})


</script>


<style scoped>
/** CSS STYLE HERE */
</style>
  