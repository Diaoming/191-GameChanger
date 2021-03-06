<template>
<v-app class="grey lighten-4">
    <Navbar />
    <v-layout row class="top-spacer">
        <v-flex md6 sm12>
            <!--100% picture are larger screen size-->
            <v-container d-none d-md-block>
                <v-img class="no-margin" height="auto" src="../assets/keyboard.png" alt="Picture of hand typing on keyboard">
                    <h1 class="tagline">Share the story of<br> your creation</h1>
                </v-img>
            </v-container>
            <!--40% picture are larger screen size-->
            <v-container d-flex d-md-none>
                <v-img height="40vh" src="../assets/keyboard.png" alt="Picture of hand typing on keyboard">
                    <h1 class="tagline">Share the story of<br> your creation</h1>
                </v-img>
            </v-container>
        </v-flex>    
        <v-layout column md7 text-center>
            <v-flex xs2 d-none d-md-block></v-flex>
            <v-flex xs1 class="mb-10 mt-6 mt-sm-0">
                <h1>Welcome Back</h1>
                <div :hidden="login_error_hidden" style="color: red">
                    <br>
                    <p>{{message}}</p>
                </div>
            </v-flex>
            <!--This is the input box for the user email-->
            <v-flex xs1>
                <v-text-field 
                    prepend-inner-icon="far fa-envelope"
                    v-model= "email"
                    text-center class="center-block input-width" 
                    label="email" 
                    placeholder="E-mail" 
                    solo>
                </v-text-field>
            </v-flex>
            <!--Add spacing when small-->
            <v-container d-md-none>
                <v-spacer></v-spacer>
            </v-container>
            <!--This is the input box for the user password-->
            <v-flex xs1>
                <v-text-field 
                    :type="'password'"
                    prepend-inner-icon="fas fa-lock" 
                    v-model= "password"
                    class="center-block input-width" 
                    label="password" 
                    placeholder="Password" 
                    solo>
                </v-text-field>
                <br>
                <router-link to="/ForgotPassword"><p><span class="underlined">Forgot password?</span></P></router-link>
            </v-flex>
            <v-flex xs2 class="mt-10">
                <!--This is the submit button that you have to bind an on-click event for-->
                 <v-flex  d-none d-md-block>
                    <v-btn height="3.5em" width="10em" color="#4DB848" class="white--text body-1" @click="loginUser">Login</v-btn>
                 </v-flex>
                 <!--Center Button for smaller screens-->
                <v-container d-flex d-md-none>
                    <br>
                    <v-btn height="3em" width="7em" color="#4DB848" class="center-block white--text body-1" @click="loginUser">Login</v-btn>
                </v-container>
                <!--Eventually, add a link to take the user to the login page when they click the span-->
                <p class="mt-6">Don't have an account yet? <router-link to="/signup"><span class="underlined">Sign up!</span></router-link></p>
            </v-flex>
        </v-layout>
    </v-layout>
    <Footer class="mt-8 mt-sm-0" />
</v-app>
</template>

<script>
import Navbar from "./Navbar";
import Footer from "./Footer";
import { Auth } from "aws-amplify";

export default {
    components: {
        Navbar,
        Footer
    },
    data() {
        return {
            email: this.$route.query.email,
            password: '',
            login_error_hidden: true,
            message: ''
        }
    },
    methods: {
        async loginUser() {
            // reset error message stuff
            this.login_error_hidden = true;

            await Auth.signIn(this.email, this.password)
                .then(user => {
                    //console.log(user)
                    this.$router.push({ path: '/homepage', query: { email: this.email } })
                    })
                .catch(err => {
                    //console.log(err);
                    this.login_error_hidden = false;
                    this.message = err.message;
                });

            var user = await Auth.currentUserInfo().then(data => {
                return data.attributes
            })

            /*
            Set up the AWS environment 
            */
            var AWS = require('aws-sdk');
            // Initialize the Amazon Cognito credentials provider
            AWS.config.region = 'us-west-2'; // Region
            AWS.config.credentials = new AWS.CognitoIdentityCredentials({
                IdentityPoolId: 'us-west-2:c8838837-ac29-45f7-b5c2-6ec245a55ed1',
            });

            var dynamodb = new AWS.DynamoDB.DocumentClient();

            // if user does not exist in user database, add them in there
            if (await this.checkIfUserExistsInDatabase(dynamodb, user.email) == false) {
                this.addUserToDatabase(dynamodb, user)
            }

            

        },

        /*
        Return false if the user does not exist in the database, otherwise return true
        */
        async checkIfUserExistsInDatabase(dynamodb, userEmail) {
            // check if user exists in database first
            var params = {
                TableName: 'user-info',
                Key: {
                    "email": userEmail
                }
            }

            // return whether the user does exist (or not)
            return await dynamodb.get(params).promise().then(user => {
                if (user.Item == undefined) {
                    return false
                }
                else {
                    return true
                }
            })
        },

        /*
        Add the user to the database if they are not in it already.
        */
        addUserToDatabase(dynamodb, user) {
            var params = {
                TableName: 'user-info',
                Item: {
                    email: user.email,
                    password: this.password,
                    displayName: user.name,
                    projects: [],
                    socialMedia: {
                        "Github": "none",
                        "Twitter": "none",
                        "LinkedIn": "none"
                    },
                    bio: '', 
                    profilePicture: 'https://gamechangerhackathonusers.s3-us-west-2.amazonaws.com/blank.png'
                }
            }

            dynamodb.put(params, function(err) {
                if (err) {
                    console.log(err)
                }
                else {
                    console.log('success')
                }
            })
        },
        
        

    }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Text+Me+One&display=swap');

.underlined {
    text-decoration: underline;
}
.input-width{
    width: 50%;
}
.center-block {
  display: block;
  margin-right: auto;
  margin-left: auto;
}

.tagline {
    font-family: 'Text Me One', sans-serif;
    text-transform: uppercase;
    color:white;
    font-weight: normal;
    margin: 8vh;
    font-size:2.5em;
}

.no-margin {
    margin-bottom: -2em;
}

.top-spacer {
    margin-top: 2.5em;
}

</style>
