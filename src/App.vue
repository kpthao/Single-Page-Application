<script>
import $ from 'jquery'
import L from 'leaflet'
import 'leaflet-control-geocoder'
import new_incident from './components/new_incident.vue'

export default {
    data() {
        return {
            view: 'map',
            codes: '',
            codes: [],
            neighborhood: '',
            neighborhoods: [],
            accidents: '',
            incidents: [],
            address: '',
            map: null,
            case_number: '', date: '', time: '', code: '', incident: '',police_grid: '', neighborhood_number: '', block: '',
            searchAddressQuery: '',
            searchLatQuery: '',
            searchLngQuery: '',
            geojsonLayer: null,
            leaflet: {
                map: null,
                center: {
                    lat: 44.955139,
                    lng: -93.102222,
                    address: ""
                },
                zoom: 12,
                bounds: {
                    nw: {lat: 45.008206, lng: -93.217977},
                    se: {lat: 44.883658, lng: -92.993787}
                },
                neighborhood_markers: [
                    {location: [44.942068, -93.020521], marker: null, onMap: true },
                    {location: [44.977413, -93.025156], marker: null, onMap: true },
                    {location: [44.931244, -93.079578], marker: null, onMap: true },
                    {location: [44.956192, -93.060189], marker: null, onMap: true },
                    {location: [44.978883, -93.068163], marker: null, onMap: true },
                    {location: [44.975766, -93.113887], marker: null, onMap: true },
                    {location: [44.959639, -93.121271], marker: null, onMap: true },
                    {location: [44.947700, -93.128505], marker: null, onMap: true },
                    {location: [44.930276, -93.119911], marker: null, onMap: true },
                    {location: [44.982752, -93.147910], marker: null, onMap: true },
                    {location: [44.963631, -93.167548], marker: null, onMap: true },
                    {location: [44.973971, -93.197965], marker: null, onMap: true },
                    {location: [44.949043, -93.178261], marker: null, onMap: true },
                    {location: [44.934848, -93.176736], marker: null, onMap: true },
                    {location: [44.913106, -93.170779], marker: null, onMap: true },
                    {location: [44.937705, -93.136997], marker: null, onMap: true },
                    {location: [44.949203, -93.093739], marker: null, onMap: true }
                ]
            },
        };
    },
    methods: {
        viewMap(event) {
            this.view = 'map';
        },

        viewNewIncident(event) {
            this.view = 'new_incident';
        },

        viewAbout(event) {
            this.view = 'about';
        },

        mapChange(event){
            this.panTo([44.955139, -93.102222])
            
            this.leaflet.center.lat= 44.955139;
            this.leaflet.center.lng = -93.102222;
        },

        neighborMarker(){
        for (let i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
            let marker = new L.Marker([this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]]);
            let num = i+1;
            let url = 'http://localhost:8000/count?neighborhood=' + num;
            this.getJSON(url).then((results) => {
            console.log(results);
            marker.bindPopup(results[0].count + " incidents occured.");
            this.leaflet.neighborhood_markers[i].marker = marker;
            marker.addTo(this.leaflet.map);
            })
        .catch((error) =>{
         alert("Marker Error:");
         console.log(error);
        })
        }
        },

        getJSON(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'json',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({status: status.status, message: status.statusText});
                    }
                });
            });
        },

        uploadJSON(method, url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    type: method,
                    url: url,
                    contentType: 'application/json; charset=utf-8',
                    data: JSON.stringify(data),
                    dataType: 'text',
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({status: status.status, message: status.statusText});
                    }
                });
            });
        },
        getDatabase(){
            let incident_url = "http://localhost:8000/incidents?" + query;
            let neighborhood_url = "http://localhost:8000/neighborhoods";
            let code_url = "http://localhost:8000/codes"
            Promise.all([this.getJSON(incident_url),this.getJSON(neighborhood_url),this.getJSON(code_url)])
            .then((results) => {
                console.log(results)
                this.incidents = JSON.parse(JSON.stringify(results[0]));            
                this.neighborhoods = JSON.parse(JSON.stringify(results[1]));
                this.codes = JSON.parse(JSON.stringify(results[2]));
            })
            .catch((error) =>{
                console.log("Error:", error);
            })
            // <!-- 
            // Promisve.all([getJSON(...), getJSON(...), getJSON(...)]) makes it so the .then does not trigger until they are all done 
            // .then((results)=>{})
            // .catch((err)=>{});
            // Ideal for the crime API we need to get data from. 
            // results will be a list, results[0] will be the first promise, 
            // results[1] will be the data with the second promise, etc...
            // this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            //     // St. Paul GeoJSON
            //     $(result.features).each((key, value) => {
            //         district_boundary.addData(value);
            // });
            // }).catch((error) => {
            //     console.log('Error:', error);
            // });
            // Code from up above
            
            // -->
        },
      /*  newAccidentSubmit(data){
            console.log("inside submit")
            $.ajax({
                type: 'PUT',
                url: 'http://localhost:8000/new-accident-submit',
                data: JSON.stringify(data),
                contentType: 'application/json; charset=utf-8',
                dataType: 'text',
                success: function(response) {
                    console.log('Record updated successfully');
                },
                error: function(xhr, status, error) {
                    console.error('Error inserting record:', error);
                }
            });
        },*/
        deleteIncident(caseNumber){
            $.ajax({
                type: 'DELETE',
                url: "http://localhost:8000/remove-incident",
                contentType: 'application/json',
                data: "{\"case_number\":"+String(caseNumber)+"}",
                success: function(response) {
                  // handle success
                  this.leaflet.map.reload()
                },
                error: function(error) {
                  // handle error
                  console.log("whoops");
                }
            });
        },
        async searchAddress(){
            //Use Nominatim to search for the location
            console.log(this.searchAddressQuery)

            // Check if the search AddressQuery is empty
            if(this.searchAddressQuery === ''){
                //if it's empty, show an error message
                alert('Please enter an address to search')
                return
            }
            L.Control.Geocoder.nominatim().geocode(this.searchAddressQuery, (results) =>
            {
                //Check if there are any results
                if(results.length > 0){
                    console.log(results)
                    //if there are, get the first result
                    const result = results[0]
                    console.log(results[0])
                    console.log(result.center.lat, result.center.lng)
                    //Set the center of the map to the latitude and longitude of the result
                    //this.current_marker = new L.Marker([result[0].lat,result[0].lon]).addTo(this.leaflet.map);
                    this.leaflet.map.setView([result.center.lat, result.center.lng], 17)
                    //Update the searchLatQuery and searchLngQuery values
                    this.searchLatQuery = result.lat
                    this.searchLngQuery = result.lng;
                    let marker = new L.Marker([this.searchLatQuery, this.searchLngQuery])
                    marker.addTo(this.leaflet.map)
                } else {
                    //if there are no results, show an error message
                    alert('No results found for the given address')
                }
            })
        },
        async searchCoordinate(){
            // Check if the searchLatQuery or searchLngQuery is empty
            if(this.searchLatQuery === '' || this.searchLngQuery === ''){
                alert('Please enter a latitude and longitude to search')
                return
            }

            // Check if the searchLatQuery is a valid latitude
            if(isNaN(this.searchLatQuery) || this.searchLatQuery < -90 || this.searchLatQuery > 90){
                // If it's not a valid latitude, show an error message
                alert('Please enter a valid latitude')
                return
            }

            if(isNaN(this.searchLngQuery) || this.searchLngQuery < -180 || this.searchLngQuery > 180){
                alert('Please enter a valid longitude')
                return
            }

            // Set the center of the map to the latitude and longitude entered
            this.leaflet.map.setView([this.searchLatQuery, this.searchLngQuery], 17)
            let marker = new L.Marker([this.searchLatQuery, this.searchLngQuery])
            marker.addTo(this.leaflet.map)
        }

    },
    //Sets up Leaflet map and adds it to the page. Also adds tile layer and GeoJSON data to the map.
    async mounted() {
        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);

        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });
        //Initialize Database on loadup
        this.getDatabase();
    },

    components: {
        new_incident
    }
}
</script>

<template>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <p :class="'cell small-4 ' + ((view === 'map') ? 'selected' : 'unselected')" @click="viewMap">Map</p>
            <p :class="'cell small-4 ' + ((view === 'new_incident') ? 'selected' : 'unselected')" @click="viewNewIncident">New Incident</p>
            <p :class="'cell small-4 ' + ((view === 'about') ? 'selected' : 'unselected')" @click="viewAbout">About</p>
        </div>
    </div>
    <div v-show="view === 'map'">
        <div class="grid-container">
            <!-- text box and search button -->
            <form id="search-form" style="padding: 1rem">
                <div class="grid-x grid-padding-x" style="border: 1px solid black">
                    <div class="cell small-12 medium-6 large-6">
                        <label for="case" style="padding-top: 1rem">Address: (ex. East 7th Street Saint Paul Minnesota)</label>
                        <input type="text" v-model="searchAddressQuery" placeholder="Please enter an address here"/>
                        <button type="submit" @click.prevent="searchAddress" class="button">Search</button>
                    </div>
                    <div class="cell small-12 medium-6 large-6">
                        <div>
                            <label for="case"  style="padding-top: 1rem">Latitude: (ex. 44.959716760336484)</label>
                            <input type="text" v-model="searchLatQuery" placeholder="Please enter latitude here"/>
                        </div>
                        <div>
                            <label for="case">Longitude: (ex. -93.20777042679448)</label>
                            <input type="text" v-model="searchLngQuery" placeholder="Please enter longitude here"/>
                        </div>
                        <button type="submit" @click.prevent="searchCoordinate" class="button">Search</button>
                    </div>
                </div>
            </form>
            <!-- actual map object -->
            <div class="grid-x grid-padding-x">
                <div id="leafletmap" class="cell auto"></div>
            </div>
            <!-- This is the stuff for making the 'incidents' table underneath the map-->
            <div class="grid-x grid-padding-x">
                <table>
                    <tr>
                        <th>Item</th>
                        <th>Case Number</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Code</th>
                        <th>Incident</th>
                        <th>Police-Grid</th>
                        <th>Neighborhood</th>
                        <th>Block</th>
                    </tr>
                    <tr v-for="(value, key) in this.incidents" :key="key">
                        <button type="button" class="button" @click="selectedCrimeMarker(value.block)" style="margin-top: 15px">Find Location</button>
                        <td>{{ value.case_number }}</td>
                        <td v-text="getType(value.code)"></td> 
                        <td>{{ value.incident }}</td>
                        <td>{{ value.date }}</td>
                        <td>{{ value.time }}</td>
                        <td>{{ value.police_grid }}</td>
                        <td v-text="getNeighborhood(value.neighborhood_number)"></td>
                        <td v-text="replaceX(value.block)"></td>
                        <td><input type="submit" value="Delete" class="button alert" @click="deleteIncident(value.case_number)"></td>
                    </tr> 
                    <!-- <tr v-for="index in 1000" :key="index">
                        <td> {{index}} #</td>
                        <td>CASE</td>
                        <td>DATE</td>
                        <td>TIME</td>
                        <td>CODE</td>
                        <td>INCIDENT</td>
                        <td>POLICE-Grid</td>
                        <td>NEIGHBORHOOD</td>
                        <td>BLOCK</td>
                    </tr> -->
                </table>
            </div>
        </div>
    </div>

    <!-- FORM INPUT STUFF-->
    <div v-if="view === 'new_incident'">
        <new_incident />
    </div>
       
    <div v-if="view === 'about'">
        <!-- Replace this with your actual about the project content: can be done here or by making a new component -->

        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell small-12 medium-12 large-12 about-the-project">About the Project</h1>

                <div class ="cell small-12 medium-4 large-4 person-box">
                    <img src="photos/KP_photo.jpg" alt="Photo of Kong Pheng Thao" width="500px" height="500px">
                    <h3 class="name-header">Kong Pheng Thao</h3>
                    <div>
                        Hello, I'm a senior at the University of St. Thomas studying computer science. I have always been interested in technology and the things you can do with it. As such, I find that computer science can bring so much leisure as well as joy in people's lives.
                    </div>
                </div>
                <div class = "cell small-12 medium-4 large-4 person-box">
                    <img src="photos/MA_photo.jpg" alt="Photo of Michael Andreucci" width="500px" height="600px">
                    <h3 class="name-header">Michael Andreucci</h3>
                    <div>
                        Hello, I am currently a senior at St. Thomas studying Computer science. I am currently planning to focus on machine learning after I graduate in the spring. I find machine learnings potential to be fascinating and want to see how much AI can improve the world. I am also interested in doing some game development but I don't know if I would prefer that over machine learning at the moment.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <img src="photos/AS_photo.png" alt="Photo of Andrew Steichen" width="500px" height="500px">
                    <h3 class="name-header">Andrew Steichen</h3>
                    <div>
                        I am a senior at St. Thomas studying Computer Science, getting better at it every day. I have found my favorite specialization to be within information security, mainly due to its direct problem-solving nature. 
                    </div>
                </div>

                <div class="cell small-12 medium-12 large-12 description">
                    <h1 class="about-the-project">Description of the Tools</h1>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <h3 class="name-header">Leaflet</h3>
                    <div class="text-center">
                        Leaflet is an open-source JavaScript library for creating interactive maps. It is designed to be lightweight and easy to use, and it is widely used for creating web-based maps for a variety of purposes, such as real estate, travel, and data visualization.
                        <br><br>
                        Leaflet is built on top of the widely used open-source mapping library, OpenLayers, and it uses a similar API. It is a flexible library that allows developers to create a wide range of maps, from simple street maps to complex interactive maps with custom markers, popups, and layers.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <h3 class="name-header">Leaflet-Control-Geocoder</h3>
                    <div class="text-center">
                        leaflet-control-geocoder is a plugin for the Leaflet JavaScript library that adds geocoding functionality to Leaflet maps. Geocoding is the process of converting an address or place name into a set of geographic coordinates (latitude and longitude), and vice versa.
                        <br><br>
                        leaflet-control-geocoder adds a search box to the map, which allows users to search for an address or place name and have the map automatically pan and zoom to that location. It uses various geocoding services (such as Nominatim and Google Maps) to perform the geocoding, and it can be customized to use a specific service or a combination of services.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <h3 class="name-header">JQuery</h3>
                    <div class="text-center">
                        jQuery is a fast, small, and feature-rich JavaScript library that makes it easy to interact with HTML documents. It is widely used to simplify client-side scripting of HTML, and it is designed to make it easier to navigate a document, select DOM elements, create animations, handle events, and develop Ajax applications.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <h3 class="name-header">Vue.JS</h3>
                    <div class="text-center">
                        Vue.js is an open-source JavaScript framework for building web applications. It is designed to be lightweight, easy to use, and flexible, and it allows developers to create single-page applications (SPAs) and reactive user interfaces.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">Foundation</h3>
                    <div class="text-center">
                        Foundation is a responsive front-end framework that allows developers to quickly and easily create responsive websites and web applications. It is designed to be lightweight, customizable, and easy to use, and it provides a range of features and tools to help developers build responsive layouts, style elements, and add functionality to their projects.
                    </div>
                </div>
                <div class="cell small-12 medium-12 large-12 description">
                    <h1 class="about-the-project">Video Demo</h1>
                </div>
                <div class="cell small-12 medium-12 large-12 person-box" style="padding-bottom: 2rem">
                    <div class="text-center">
                        video link 
                    </div>
                </div>
                <div class="cell small-12 medium-12 large-12 description">
                    <h1 class="about-the-project">6 Interesting Things</h1>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">1</h3>
                    <div class="text-center">
                        Something I found interesting to see were the various crime "hotspots" where many of the listed crimes occured such as in Capitol River.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">2</h3>
                    <div class="text-center">
                        It was interesting to see how the time of year affected the amount and the type of crimes commited.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">3</h3>
                    <div class="text-center">
                        See the total amount of crimes commited since August of 2014 was really shocking. I knew it would be a lot but actually visualising how many was interesting.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">4</h3>
                    <div class="text-center">
                        It was interesting to see the breakdown of what types of crime were commited. It seems like property crime was the most prevalent.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">5</h3>
                    <div class="text-center">
                        Not so much about the data itself but it was interesting to see how the map updates when paning and zooming on incidents.
                    </div>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box" style="padding-bottom: 2rem">
                    <h3 class="name-header">6</h3>
                    <div class="text-center">
                        Lastly I found it interesting to see how few incidents occured near summit hill compared to other places.
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style>
#leafletmap {
    height: 600px;
}

.selected {
    background-color: rgb(10, 100, 126);
    color: white;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
.unselected {
    background-color: rgb(200, 200, 200);
    color: black;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
table, th, td{
    text-align:center;
    color: black;
    border: 1px solid black;
    background-color:darkgrey;
}
.about-the-project{
    padding: 2rem;
    display: flex;
    justify-content: center;
    border: 1px solid black;
}
.new-incident-form{
    padding: 2rem;
    display: flex;
    justify-content: center;
    border: 1px solid black;
}
.person-box{
    border: 1px solid black;
}
.name-header{
    padding-top: 1rem;
    display: flex;
    justify-content: center;
    font-weight: bold;
    text-decoration: underline;
    text-align: center;
}
.description{
    padding-top: 2rem;
    display: flex;
    justify-content: center;
}
.text-center{
    text-align: center;
    /* padding-top: 1rem; */
    padding-bottom: 1rem;
}
</style>
