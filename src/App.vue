<script>
import $ from 'jquery'
import L from 'leaflet'
import 'leaflet-control-geocoder'

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [],
            incidents: [],
            address: '',
            map: null,
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
                    {location: [44.942068, -93.020521], marker: null},
                    {location: [44.977413, -93.025156], marker: null},
                    {location: [44.931244, -93.079578], marker: null},
                    {location: [44.956192, -93.060189], marker: null},
                    {location: [44.978883, -93.068163], marker: null},
                    {location: [44.975766, -93.113887], marker: null},
                    {location: [44.959639, -93.121271], marker: null},
                    {location: [44.947700, -93.128505], marker: null},
                    {location: [44.930276, -93.119911], marker: null},
                    {location: [44.982752, -93.147910], marker: null},
                    {location: [44.963631, -93.167548], marker: null},
                    {location: [44.973971, -93.197965], marker: null},
                    {location: [44.949043, -93.178261], marker: null},
                    {location: [44.934848, -93.176736], marker: null},
                    {location: [44.913106, -93.170779], marker: null},
                    {location: [44.937705, -93.136997], marker: null},
                    {location: [44.949203, -93.093739], marker: null}
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

        viewAddressSubmission(event) {
            this.view = 'address'
        },

        viewCoordinateSubmission(event) {
            this.view = 'latitude-and-longitude'
        },

        mapChange(event){
            this.panTo([44.955139, -93.102222])
            
            this.leaflet.center.lat= 44.955139;
            this.leaflet.center.lng = -93.102222;
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
        async searchAddress(){
            //Use Nominatim to search for the location
            // 44.959716760336484,-93.20777042679448
            console.log(this.searchAddressQuery)

            // Check if the search AddressQuery is empty
            if(this.searchAddressQuery === ''){
                //if it's empty, show an error message
                alert('Please enter an address to search')
                return
            }
            // const geocoder = L.control.geocoder('nominatim').addTo(this.leaflet.map)
            // const geocoder = new L.Control.Geocoder().addTo(this.leaflet.map)
            //Use Leaflet's geocoding plugin to search for the address
            // geocoder.Geocoder.Geocode(this.searchAddressQuery, (results) =>
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
                    this.leaflet.map.setView([result.center.lat, result.center.lng], 17)
                    //Update the searchLatQuery and searchLngQuery values
                    this.searchLatQuery = result.lat
                    this.searchLngQuery = result.lng;
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
        }
    },
    //Sets up Leaflet map and adds it to the page. Also adds tile layer and GeoJSON data to the map.
    mounted() {
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
                        <label for="case" style="padding-top: 1rem">Address:</label>
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
                <button type="button" @click="mapChange">TEST</button>
                <div>____</div>
                <button type="button">TEST2</button>
                <div>____</div>
                <p @click="mapChange">TEST3</p>
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
                    <tr v-for="index in 1000" :key="index">
                        <td> {{index}} #</td>
                        <td>CASE</td>
                        <td>DATE</td>
                        <td>TIME</td>
                        <td>CODE</td>
                        <td>INCIDENT</td>
                        <td>POLICE-Grid</td>
                        <td>NEIGHBORHOOD</td>
                        <td>BLOCK</td>
                    </tr>
                </table>
            </div>
        </div>
    </div>
    <!-- FORM INPUT STUFF-->
    <div v-if="view === 'new_incident'">
        <!-- Replace this with your actual form: can be done here or by making a new component -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell small-12 medium-12 large-12 new-incident-form">New Incident Form</h1>
            </div>
        </div>
        <form>
            <div class="grid-container" style='border: 1px solid black'>
                <div class="grid-x grid-padding-x">
                    <div class="cell small-12 medium-6 large-4">
                        <label for="case">Case Number:</label>
                        <input type="text" id="case" name="case">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Date">Date (YYYY-MM-DD):</label>
                        <input type="text" id="Date" name="Date">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Time">Time (HR:MM:SS):</label>
                        <input type="text" id="Time" name="Time">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Code">Code:</label>
                        <input type="text" id="Code" name="Code">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Incident">Incident</label>
                        <input type="text" id="Incident" name="Incident">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="PGrid">Police Grid</label>
                        <input type="text" id="PGrid" name="PGrid">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Neighbor">Neighborhood Name:</label>
                        <input type="text" id="Neighbor" name="Neighbor">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <label for="Street">Block/Street:</label>
                        <input type="text" id="Street" name="Street">
                    </div>
                    <div class="cell small-12 medium-6 large-4">
                        <input type="button" value="SUBMIT" class="button">
                    </div>
                </div>
            </div>
        </form>
    </div>
    <div v-if="view === 'about'">
        <!-- Replace this with your actual about the project content: can be done here or by making a new component -->
        <!-- 
            Promisve.all([getJSON(...), getJSON(...), getJSON(...)]) makes it so the .then does not trigger until they are all done 
            .then((results)=>{})
            .catch((err)=>{});
            Ideal for the crime API we need to get data from. 
            results will be a list, results[0] will be the first promise, 
            results[1] will be the data with the second promise, etc...
        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });
        Code from up above
            
        -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell small-12 medium-12 large-12 about-the-project">About the Project</h1>

                <div class ="cell small-12 medium-4 large-4 person-box">
                    <img src="photos/KP_photo.jpg" alt="Photo of Kong Pheng Thao">
                    <h3 class="name-header">Kong Pheng Thao</h3>
                    <div>
                        Hello, I'm a senior at the University of St. Thomas studying computer science. I have always been interested in technology and the things you can do with it. As such, I find that computer science can bring so much leisure as well as joy in people's lives.
                    </div>
                </div>
                <div class = "cell small-12 medium-4 large-4 person-box">
                    <h3>Michael Andreucci</h3>
                </div>
                <div class="cell small-12 medium-4 large-4 person-box">
                    <h3>Andrew Steichen</h3>
                </div>
                <div class="cell small-12 medium-12 large-12 description">
                    <h1>Description of the Tools</h1>
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
    display: flex;
    justify-content: center;
}
.description{
    padding-top: 2rem;
    display: flex;
    justify-content: center;
}
</style>
