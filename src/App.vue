<template>
  <div id="app">
    <audio ref="remoteAudio" controls>
      <p>Your browser doesn't support HTML5 audio.</p>
    </audio>
    <br>
    <br>
    <!-- <button @click="registerUserAgent">Register</button>
    <br> -->
   
    <br>
  
    <button @click="echo_call">Echo Test Call</button>
     <br>
     <br>
     <button @click="conf_call">Conference Call</button>
     <br>
     <br>
     <button @click="queue_call">Queue Call</button>
     <br>
     <br>
    <input type="text" v-model="dtmf">
    <button @click="sendDtmf">Send DTMF</button>
    <br> 
    <br> 

    <button @click="endCall">Hangup</button>

  </div>
</template>

<script>
import { UserAgent, Registerer, Inviter, SessionState } from "sip.js";

export default {
  name: 'App',
  components: {
  },

  data () {
    return {
       userAgent: null,
       phone_number: "123",
     //  mediaElement: this.$refs.remoteAudio,
       inviter: null,
       dtmf: "",
    }
},

  async created () {
    this.registerUserAgent()


    //
    // const aor= "sip:webrtc_client@naqi.marayshi.com";
    // const options= {
    // aor,
    // password: "83745897345",
    // authorizationUser: "webrtc_client",
    // traceSip: true,
    //   media: {
    //     constraints: { audio: true, video: false }, // audio only call
    //     remote: {
    //       audio: document.getElementById('remoteAudio')
    //     }
    //   }
    // };
    // const simpleUser = new Web.SimpleUser(this.server, options);  
    // await simpleUser.connect();
    // await simpleUser.register();

  
//    const uri = UserAgent.makeURI("sip:webrtc_client@naqi.marayshi.com");

    // const userAgentOptions = {
      
    // };
      

    //   userAgent.start().then(() => {
    //   const target = UserAgent.makeURI("sip:6002@178.62.70.22:5070");

    //   const inviter = new Inviter(userAgent, target);
    //   inviter.invite({
    //     earlyMedia: true
    //   });
    // });

    },

  computed:{
    
  },

  beforeMount() {
    window.addEventListener("beforeunload", this.preventNav)
  },

  beforeDestroy() {
    window.removeEventListener("beforeunload", this.preventNav);
  },

  methods:{
    registerUserAgent () {
      const transportOptions = {
            server: "wss://naqi.marayshi.com:8089/ws",
            traceSip: true,
          };
        const uri = UserAgent.makeURI("sip:webrtc_client@naqi.marayshi.com");
      const userAgentOptions = {
        authorizationPassword: '83745897345',
        authorizationUsername: 'webrtc_client',
        transportOptions,
        uri
      };
     const userAgent = new UserAgent(userAgentOptions);

      const registerer = new Registerer(userAgent,{
         ice: true, // RFC 5768
      });
      userAgent.start().then(() => {
        registerer.register();
      });

      this.userAgent = userAgent;

      
    },
    echo_call(){

     // this.userAgent.start().then(() => {
      const target = UserAgent.makeURI("sip:echo@naqi.marayshi.com");

       this.inviter = new Inviter(this.userAgent, target, {
        earlyMedia: true,
        sessionDescriptionHandlerOptions: {
          constraints: {
            audio: true,
            video: false
          }
        }
      });

      this.inviter.stateChange.addListener((state) => {
        console.log(`Session state changed to ${state}`);

         switch (state) {
          case SessionState.Initial:
            break;
          case SessionState.Establishing:
            break;
          case SessionState.Established:
            this.setupRemoteMedia(this.inviter);
            break;
          case SessionState.Terminating:
            // fall through
          case SessionState.Terminated:
            this.cleanupMedia();
            break;
          default:
            throw new Error("Unknown session state.");
         }
      })

      this.inviter.invite();
  //  });
    },

    conf_call(){

     // this.userAgent.start().then(() => {
      const target = UserAgent.makeURI("sip:conf@naqi.marayshi.com");

       this.inviter = new Inviter(this.userAgent, target, {
        earlyMedia: true,
        sessionDescriptionHandlerOptions: {
          constraints: {
            audio: true,
            video: false
          }
        }
      });

      this.inviter.stateChange.addListener((state) => {
        console.log(`Session state changed to ${state}`);

         switch (state) {
          case SessionState.Initial:
            break;
          case SessionState.Establishing:
            break;
          case SessionState.Established:
            this.setupRemoteMedia(this.inviter);
            break;
          case SessionState.Terminating:
            // fall through
          case SessionState.Terminated:
            this.cleanupMedia();
            break;
          default:
            throw new Error("Unknown session state.");
         }
      })

      this.inviter.invite();
  //  });
    },

queue_call(){

     // this.userAgent.start().then(() => {
      const target = UserAgent.makeURI("sip:queue@naqi.marayshi.com");

       this.inviter = new Inviter(this.userAgent, target, {
        earlyMedia: true,
        sessionDescriptionHandlerOptions: {
          constraints: {
            audio: true,
            video: false
          }
        }
      });

      this.inviter.stateChange.addListener((state) => {
        console.log(`Session state changed to ${state}`);

         switch (state) {
          case SessionState.Initial:
            break;
          case SessionState.Establishing:
            break;
          case SessionState.Established:
            this.setupRemoteMedia(this.inviter);
            break;
          case SessionState.Terminating:
            // fall through
          case SessionState.Terminated:
            this.cleanupMedia();
            break;
          default:
            throw new Error("Unknown session state.");
         }
      })

      this.inviter.invite();
  //  });
    },
    
    setupRemoteMedia(session){
      const remoteStream = new MediaStream();
        session.sessionDescriptionHandler.peerConnection.getReceivers().forEach((receiver) => {
        if (receiver.track) {
            remoteStream.addTrack(receiver.track);
          }
        });
        this.$refs.remoteAudio.srcObject = remoteStream;
        this.$refs.remoteAudio.play();
    },
    cleanupMedia(){
      this.$refs.remoteAudio.srcObject = null;
      this.$refs.remoteAudio.pause();
    },

    endCall() {
      const session = this.inviter
      switch(session.state) {
        case SessionState.Initial:
        case SessionState.Establishing:
          session.cancel();
          // if (session instanceOf Inviter) {
          //   // An unestablished outgoing session
          //   session.cancel();
          // } else {
          //   // An unestablished incoming session
          //   session.reject();
          // }
          this.cleanupMedia();
          break;
        case SessionState.Established:
          // An established session
          session.bye();
          this.cleanupMedia();
          break;
        case SessionState.Terminating:
        case SessionState.Terminated:
          // Cannot terminate a session that is already terminated
      break;
      default:
            session.cancel();
            this.cleanupMedia();
    }
  },

  sendDtmf(){
    const session = this.inviter
    const options = {
      requestOptions: {
        body: {
          contentDisposition: "render",
          contentType: "application/dtmf-relay",
          content: "Signal="+this.dtmf+"\r\nDuration=1000"
        }
      }
    };
    session.info(options)
  },

  preventNav( ) {
      // event.preventDefault()
      // 
      this.endCall()
      //event.returnValue = ""
    }
  },

  // beforeDestroy(){
  //   this.endCall()
  //   this.cleanupMedia()
  // },

  // destroyed(){
  //    this.endCall()
  //   this.cleanupMedia()
  // }


}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
