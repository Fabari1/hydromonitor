<template>
    <v-container fluid color="surface" align="center">
        <v-row  max-width="1200px">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container"></div>

                </figure>
                
            </v-col>
            <v-col cols="3">
                <v-card  margin-bottom="5" max-width="500" color="primaryContainer" subtitle="Temperature">
                    <v-card-item> 
                        <span class="text-h3"> {{ temperature }} </span>
                    </v-card-item>
                </v-card>
                <v-card  margin-bottom="5" max-width="500" color="tertiaryContainer" subtitle="Heat Index (Feels Like)">
                    <v-card-item> 
                        <span class="text-h3"> {{ heatindex }} </span>
                    </v-card-item>
                </v-card>
                <v-card  margin-bottom="5" max-width="500" color="secondaryContainer" subtitle="Humidity">
                    <v-card-item> 
                        <span class="text-h3"> {{ humidity }} </span>
                    </v-card-item>
                </v-card>
            </v-col>
        </v-row>
        <v-row max-width="1200px" justify="start">
            <v-col cols="9">
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
// Highcharts, Load the exporting module and Initialize exporting module.
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);


import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { storeToRefs } from "pinia";
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const Mqtt = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);
const mqtt = ref(null);
const host = ref("www.yanacreations.com"); // Host Name or IP address
const port = ref(9002); // Port number
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.
const temperature = computed(()=>{
    if(!!payload.value){
    return `${payload.value.temperature.toFixed(2)} °C`;
    }
});
const heatindex = computed(()=>{
    if(!!payload.value){
    return `${payload.value.heatindex.toFixed(2)} °C`;g
    }
});
const humidity = computed(()=>{
    if(!!payload.value){
    return `${payload.value.humidity.toFixed(2)} %`;
    }
});
const tempChart = ref(null); // Chart object
const HiChart = ref(null); // Chart object



const CreateCharts = async () => {
// TEMPERATURE CHART
    tempChart.value = Highcharts.chart('container', {
    chart: { zoomType: 'x' },
    title: { text: 'Temperature Analysis (Live)', align: 'left' },
    yAxis: {
    title: { text: '°C' , style:{color:'#000000'}},
    labels: { format: '{value} °C' }
    },
    xAxis: {
    type: 'datetime',
    title: { text: '', style:{color:'#000000'} },
    },
    tooltip: { shared:true, },
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
    } ],
    });
    
    HiChart.value = Highcharts.chart('container1', {
    chart: { zoomType: 'x' },
    title: { text: 'Humidity Analysis (Live)', align: 'left' },
    yAxis: {
    title: { text: '°C' , style:{color:'#000000'}},
    labels: { format: '{value} °C' }
    },
    xAxis: {
    type: 'datetime',
    title: { text: '', style:{color:'#000000'} },
    },
    tooltip: { shared:true, },
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
    } ],
    });
    

};



// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();

    Mqtt.connect(); // Connect to Broker located on the backend
    setTimeout( ()=>{
    // Subscribe to each topic
    Mqtt.subscribe("620154701");
    Mqtt.subscribe("620154701_pub");}, 3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});

watch(payload,(data)=> {
        if(points.value > 0){ points.value -- ; }
        else{ shift.value = true; }
        tempChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 },
        true, shift.value);
        HiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 },
        true, shift.value);
})
</script>


<style scoped>
/** CSS STYLE HERE */
Figure {
border: 2px solid black;
}


</style>
  