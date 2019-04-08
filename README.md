# Andromega (client app)
> Playing with Speech to Text/Text to Speech using webkitSpeechRecognition & IBM' Watson.

## What is it

This is a Voice+Audio chatroom experiment (Chrome Only) that listens to the user's input, saves it then reads it back to it.

Medium term goal is to have it use socket.io and let users talk to each other without having to type of read.

**[DEMO](https://admiring-davinci-5552ea.netlify.com/)**

### Specs:

* [Nuxt.Js](https://nuxtjs.org), SPA Mode
* [webkitSpeechRecognition](https://developers.google.com/web/updates/2013/01/Voice-Driven-Web-Apps-Introduction-to-the-Web-Speech-API)
* [IBM's Watson Text to Speech](https://cloud.ibm.com/docs/services/text-to-speech?topic=text-to-speech-gettingStarted&cm_mc_uid=55176413049715546420225&cm_mc_sid_50200000=90202191554736918224&cm_mc_sid_52640000=81512721554736918237#gettingStarted)
* Storage on [jsonbin.io](jsonbin.io)
* Microphone icons  by [Smashicons](https://www.flaticon.com/authors/smashicons) from [Flaticon](https://www.flaticon.com/) licensed by [CC 3.0 BY](http://creativecommons.org/licenses/by/3.0/)

#### Issues
* *audio* will sometimes randomly throw a DOMException error when attempting to play;
* recognition will stop after some interval - use the Microphone button next to Send to re-activate speech recognition

## Build Setup

``` bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm start

# generate static project
$ npm run generate
```
