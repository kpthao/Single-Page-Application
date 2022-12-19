<script>
import $ from 'jquery'
export default {
    data() {
        return {
            case_number: "",
            code:"" ,
            incident: "",
            date: "",
            time: "",
            police_grid: "",
            neighborhood_number: "",
            block: ""
        }
    },
    methods: {
        getJSON(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: "json",
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },
        incidentForm() {
            $.ajax({
                type: 'PUT',
                url: 'http://localhost:8000/new-accident-submit',
                contentType: 'application/json',
                data: {
                    "case_number": parseInt(this.case_number),
                    "code": parseInt(this.code),
                    "incident": this.incident,
                    "date": this.date,
                    "time": this.time,
                    "police_grid": parseInt(this.police_grid),
                    "neighborhood_number": parseInt(this.neighborhood_number),
                    "block": this.block},
                
                success: function(response) {
                  alert("Incident added");
                },
                error: function (xhr, thrownError) {
                    alert(thrownError);
                }
            });
        }
    }   
}

</script>

<template>

    <!-- Replace this with your actual form: can be done here or by making a new component -->
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <h1 class="cell small-12 medium-12 large-12 new-incident-form">New Incident Form</h1>

            <form class="small-12 cell" id="formSubmit" @submit="incidentForm" >
                
                <div class= "grid-x grid-padding-x">
                    <div class="cell medium-12 medium-6 large-4">
                        <label for="case">Case Number:</label>
                        <input type="text" id="case" name = "case" v-model="Case" required/>
                    </div>
                 

                <div class="cell small-12 medium-6 large-4">
                    <label for="code">Code:</label>
                    <input type="number" id="code" v-model="code" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="incident">Incident:</label>
                    <input type="text" id="incident" v-model="incident" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="date">Date (YYYY-MM-DD):</label>
                    <input type="text" id="date" name="date" v-model="date" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="Time">Time (HR:MM:SS):</label>
                    <input type="text" id="Time" name="Time" v-model="time" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="police_grid">Police Grid</label>
                    <input type="text" id="police_grid" name="police_grid" v-model="police_grid" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="neighborhood_number">Neighborhood Number:</label>
                    <input type="text" id="neighbor_number" name ="neighborhood_number" v-model="neighborhood_number" required/>
                </div>

                <div class="cell small-12 medium-6 large-4">
                    <label for="block">Block/Street:</label>
                    <input type="text" id="block" name="block" v-model="block" required/>
                </div>
                <div class="cell small-12 medium-6 large-4">
                    <input type="submit" value="SUBMIT" class="button" @click.prevent="incidentForm">
                </div>
                </div>
            </form>
        </div>
    </div>
</template>

<style>

</style>