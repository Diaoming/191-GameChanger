<template>
<v-app class="grey lighten-4">
    <Navbar />
        <!-- Main image for Events -->
        <v-img class="background top-spacer" src="../assets/events.png" alt="Concert with lights">
            <v-container fill-height align="center">
                <v-col>
                    <v-row justify="center">
                        <h1 class="white--text event-title">DreamHack Anaheim</h1>
                    </v-row>
                    <v-row justify="center">
                        <h3 class="white--text event-subtitles">Anaheim Convention Center, CA</h3>
                    </v-row>
                    <v-row justify="center">
                        <h3 class="white--text event-subtitles">February 21 - 23</h3>
                    </v-row>
                </v-col>
            </v-container>
        </v-img>
    <!-- Display event cards -->
    <v-container class="px-12 mt-6">
        <v-row>
            <h1 class="px-3 pt-0 ml-2">Events</h1>
        </v-row>
        <v-row class="d-flex justify-sm-center justify-md-space-between">
            <v-col class="pa-5" cols="12" sm="8" md="6" justify="center" v-for="(event, index) in events" :key="index">
                <v-card outlined>
                    <v-img src="../assets/event-card2.png" alt="Guy wearing headphones in front of a computer.">
                    <v-container fluid fill-height align="center" justify="center">
                        <v-card-text class="center-text">
                                    <h2 class="text-center white--text">{{event.name.S}}</h2>
                                    <p class="text-center white--text">{{event.location.S}}</p>
                                    <p class="text-center white--text">{{event.date.S}}</p>
                                    <v-btn class="body-2 card-button" color="white" outlined v-on:click=goToEventPage(event.eventLink.S)>Learn More</v-btn>
                                    </v-card-text>
                        </v-container>
                    </v-img>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
    <Footer />
</v-app>
</template>

<script>
import Navbar from "./Navbar";
import Footer from "./Footer";

export default {
    components: {
        Navbar,
        Footer
    },

    data() {
        return {
            events: {},
            location: 'Dallas',
            place: 'Kay Bailey Hutchinson Convention Center'
        }
    },

    // runs when the page is created
    created() {
        // setting up AWS environment
        var AWS = require("aws-sdk");
        // Initialize the Amazon Cognito credentials provider
        AWS.config.region = 'us-west-2'; // Region
        AWS.config.credentials = new AWS.CognitoIdentityCredentials({
            IdentityPoolId: 'us-west-2:c8838837-ac29-45f7-b5c2-6ec245a55ed1',
        });
        
        // create the dynambodb object to call dynamodb functions
        var dynamodb = new AWS.DynamoDB({apiVersion: "2012-08-10"}); 

        // pick which database table to draw data from
        var params = {
            TableName: "Events"
        }

        // get games from the projects database
        dynamodb.scan(params).promise().then(events_database => {
            //store games into games list
            this.events = events_database.Items;
        })
    }, 

    methods: {
        goToEventPage(link) {
            //console.log(link)
            window.open(link)
        }
    }

}
</script>

<style scoped>

    .event-title {
        font-size: 3.5vw;
    }

    .event-subtitles {
        font-size: 2vw;
        font-weight: 400;
    }

    .background {
        background-position: center;
        background-repeat: no-repeat;
        background-size: contain;
        
    }

    .card-button {
        display: block;
        margin-left: auto;
        margin-right: auto;
    }

    .top-spacer {
        margin-top: 3.5em;
    }
    
</style>>
