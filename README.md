# WebRtc Vue.js x Firebase

## Overview
Easy implementation of real-time communication using WebRTC by using Firebase

### Demo URL
https://webrtcqiita.web.app/

### Requirement
* Vue: 2.6.11
* Firebase: 8.1.1

### Project setup
```
$ git clone git@github.com:hond0413/webrtc.git
$ cd webrtcqiita
$ npm install
```

### Running the project
* You will need to create an .env file under the project directory.
* The .env file should contain the following information
```
VUE_APP_API_KEY=*************
VUE_APP_AUTH_DOMAIN=*************
VUE_APP_DATABASE_URL=*************
VUE_APP_PROJECT_ID=*************
VUE_APP_STORAGE_BUCKET=*************
VUE_APP_MESSAGING_SENDER_ID=*************
VUE_APP_ID=*************
VUE_APP_MEASUREMENT_ID=*************
```
```
$ npm run serve
```