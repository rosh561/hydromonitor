<template>
    <v-container class="container" fluid justify="center" align="center">
        <v-row class="row" justify="center" max-width="1200">
            <v-col class="col col1" align="center">
                <v-sheet class="rounded-t-lg mb-1" color="surface" elevation="0" max-width="800" width="100%">
                    <VCard class="text-secondary" title="LED Controls" color="surface" subtitle="Recent settings"
                        variant="tonal" flat>

                    </VCard>
                </v-sheet>
                <v-sheet class="mb-1" color="surface" elevation="0" max-width="800" width="100%">
                    <VCard class="pt-5" color="surface" variant="tonal">
                        <v-slider class="pt-2 bg-surface" append-icon="mdi:mdi-car-light-high" density="compact"
                            thumb-size="16" color="secondary" label="Brightness" v-model="led.brightness"
                            direction="horizontal" min="0" max="250" step="10" show-ticks thumb-label="always"> </v-slider>

                    </VCard>
                </v-sheet>
                <v-sheet class="mb-1" justify="center" align="center" color="surface" elevation="0" max-width="800"
                    width="100%">
                    <VCard class="pt-5" color="surface" variant="tonal">
                        <v-slider class="pt-2 bg-surface" append-icon="mdi:mdi-led-on" density="compact" thumb-size="16"
                            label="LED Nodes" v-model="led.nodes" direction="horizontal" min="1" max="7" step="1" show-ticks
                            thumb-label="always"> </v-slider>
                    </VCard>
                </v-sheet>
                <v-sheet class="mb-1 pa-2" justify="center" align="center" border color="surface" elevation="0"
                    max-width="800" width="100%">
                    <v-progress-circular :model-value="led.nodes * 15" :size="200" :width="15" :rotate="0"
                        :color="indicatorColor">
                        <span class="text-onSurface font-weight-bold">{{ led.nodes }} LED(s)</span>
                    </v-progress-circular>
                </v-sheet>
            </v-col>
            <v-col class="col col2" align="center">
                <v-color-picker v-model="led.color"></v-color-picker>
            </v-col>
        </v-row>

    </v-container>
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

const led = reactive({ "brightness": 255, "nodes": 1, "color": { r: 255, g: 0, b: 255, a: 1 } });
let timer, ID = 1000;

// COMPUTED PROPERTIES
const indicatorColor = computed(() => {
    return `rgba(${led.color.r},${led.color.g},${led.color.b},${led.color.a})`
})



// FUNCTIONS
onMounted(() => {
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect(); // Connect to Broker located on the backend
    setTimeout(() => {
        // Subscribe to each topic
        Mqtt.subscribe("620142646");
        Mqtt.subscribe("620142646_sub");
    }, 3000);
});



onBeforeUnmount(() => {
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    // unsubscribe from all topics
    Mqtt.unsubcribeAll();
})

// WATCHERS 
watch(led, (controls) => {
    clearTimeout(ID);
    ID = setTimeout(() => {
        const message =
            JSON.stringify({ "type": "controls", "brightness": controls.brightness, "leds": controls.nodes, "color": controls.color });
        Mqtt.publish("620142646_sub", message); // Publish to a topic subscribed to by the hardware 
    }, 1000)
})



</script>


<style scoped>/** CSS STYLE HERE */</style>

