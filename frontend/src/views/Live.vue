<template>
    <v-container fluid class="bg-surface" align="center">
        <v-row class="row1" max-width="1200">
            <v-col class="col1" cols="9" >
                <figure class="highcharts-figure">
                    <div id="container"></div>
                </figure>
            </v-col>
            <v-col class="col2" cols="3">
                <VCard class="mb-5" max-width="500" color="primaryContainer" subtitle="Temperature">
                    <v-card-item>
                        <span class="text-onPrimaryContainer text-h3">{{ temperature }}</span>
                    </v-card-item>
                </VCard>
                <VCard class="mb-5" max-width="500" color="tertiaryContainer" subtitle="Heat Index (Feels like)">
                    <v-card-item>
                        <span class="text-onTertiaryContainer text-h3">{{ heatindex }}</span>
                    </v-card-item>
                </VCard>
                <VCard class="mb-5" max-width="500" color="secondaryContainer" subtitle="Humidity">
                    <v-card-item>
                        <span class="text-onSecondaryContainer text-h3">{{ humidity }}</span>
                    </v-card-item>
                </VCard>
            </v-col>
        </v-row>
        <v-row class="row2" justify="start" max-width="1200">
            <v-col class="col1" cols="9">
                <figure class="highcharts-figure">
                    <div id="container1"></div>
                </figure>
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

// Highcharts, Load the exporting module and Initialize exporting module.
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);







// VARIABLES
const router = useRouter();
const route = useRoute();

const Mqtt = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);

const tempHiChart = ref(null); // Chart object
const HumidityChart = ref(null); // Chart object

const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.

const temperature = computed(()=>{
if(!!payload.value){
return `${payload.value.temperature.toFixed(2)} °C`;
}
});

const heatindex = computed(() => {
    if (!!payload.value) {
        return `${payload.value.heatindex.toFixed(2)} °C`;
    }
});

const humidity = computed(() => {
    if (!!payload.value) {
        return `${payload.value.humidity.toFixed(2)} %`;
    }
});




// FUNCTIONS
const CreateCharts = async () => {
    // TEMPERATURE CHART
    tempHiChart.value = Highcharts.chart('container', {
        chart: { zoomType: 'x' },
        title: { text: 'Temperature Analysis (Live)', align: 'left' },
        yAxis: {
            title: { text: 'Air Temperature & Heat Index', style: { color: '#000000' } },
            labels: { format: '{value} °C' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: { shared: true, },
        series: [
            {
                name: 'Temperature',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
            },
            {
                name: 'Heat Index',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[1]
            }],
    });

    // Humidity CHART
    HumidityChart.value = Highcharts.chart('container1', {
        chart: { zoomType: 'x' },
        title: { text: 'Humidity Analysis (Live)', align: 'left' },
        yAxis: {
            title: { text: 'Humidity', style: { color: '#000000' } },
            labels: { format: '{value} %' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: { shared: true, },
        series: [
            {
                name: 'Humidity',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[1]
            },
        ],
    });

};

onMounted(() => {
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();

    Mqtt.connect(); // Connect to Broker located on the backend

    setTimeout(() => {
        // Subscribe to each topic
        Mqtt.subscribe("620142646");
        Mqtt.subscribe("topic2");
    }, 3000);
});



onBeforeUnmount(() => {
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    // unsubscribe from all topics
    Mqtt.unsubcribeAll();
})

// WATCHERS
watch(payload, (data) => {
    if (points.value > 0) { points.value--; }
    else { shift.value = true; }
    tempHiChart.value.series[0].addPoint({ y: parseFloat(data.temperature.toFixed(2)), x: data.timestamp * 1000 },
        true, shift.value);
    tempHiChart.value.series[1].addPoint({ y: parseFloat(data.heatindex.toFixed(2)), x: data.timestamp * 1000 },
        true, shift.value);
    HumidityChart.value.series[0].addPoint({ y: parseFloat(data.humidity.toFixed(2)), x: data.timestamp * 1000 },
        true, shift.value);
})



</script>


<style scoped>
/** CSS STYLE HERE */
Figure {
    border: 2px solid black;
}
</style>
  