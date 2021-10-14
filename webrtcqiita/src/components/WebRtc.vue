<template>
  <div>
    <v-row>
      <v-col>
        <video class="media_v" id="webcamVideo" autoplay playsinline></video>
      </v-col>
      <v-col>
        <video class="media_v" id="remoteVideo" autoplay playsinline></video>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import firebase from "firebase"
import "firebase/firestore"

export default {
  name: "Webrtc",
  data () {
    return {
      db: null,
      servers: {
        iceServers: [
          {
            urls: [
              "stun:stun1.l.google.com:19302",
            ],
          },
        ],
        iceCandidatePoolSize: 10,
      },
      localStream: null,
      remoteStream: null,
      pc: null,
      webcamVideo: null,
      remoteVideo: null,
    }
  },
  async created() {
    this.db = firebase.firestore()
    await this.startWebcam();
    const name = this.$route.query.name
    if (name) {
      await this.createCall();
    } else {
      await this.answerBtn(this.$route.query.id);
    }
  },
  mounted() {
    this.pc = new RTCPeerConnection(this.servers);
    // HTML elements
    this.webcamVideo = document.getElementById("webcamVideo");
    this.remoteVideo = document.getElementById("remoteVideo");
  },
  methods: {
    async startWebcam() {
      // media device の設定
      this.localStream = await navigator.mediaDevices.getUserMedia({
        video: true,
        // audio: true,
      }).
      catch(error => {
        console.error("can't connect mediadevices")
      })

      this.remoteStream = new MediaStream();

      // peer connectionに local streamのtrackをセット
      this.localStream.getTracks().forEach((track) => {
        this.pc.addTrack(track, this.localStream);
      });

      // Pull tracks from remote stream, add to video stream
      this.pc.ontrack = (event) => {
        console.log(event)
        event.streams[0].getTracks().forEach((track) => {
          this.remoteStream.addTrack(track);
        });
      };

      this.webcamVideo.srcObject = this.localStream;
      this.remoteVideo.srcObject = this.remoteStream;
    },
    async createCall() {
      // Reference Firestore collections for signaling
      const callDoc = this.db.collection("calls").doc();
      const offerCandidates = callDoc.collection("offerCandidates");
      const answerCandidates = callDoc.collection("answerCandidates");

      // Get candidates for caller, save to db
      this.pc.onicecandidate = (event) => {
        event.candidate && offerCandidates.add(event.candidate.toJSON());
      };

      // Create offer
      const offerDescription = await this.pc.createOffer();
      await this.pc.setLocalDescription(offerDescription);

      const offer = {
        sdp: offerDescription.sdp,
        type: offerDescription.type,
      };

      await callDoc.set({offer});
      await callDoc.update({ name: this.$route.query.name });

      // Listen for remote answer
      callDoc.onSnapshot((snapshot) => {
        const data = snapshot.data();
        if (!this.pc.currentRemoteDescription && data?.answer) {
          const answerDescription = new RTCSessionDescription(data.answer);
          this.pc.setRemoteDescription(answerDescription);
        }
      });

      // When answered, add candidate to peer connection
      answerCandidates.onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
          if (change.type === "added") {
            const candidate = new RTCIceCandidate(change.doc.data());
            this.pc.addIceCandidate(candidate);
          }
        });
      });
    },
    async answerBtn(val) {
      const callDoc = this.db.collection("calls").doc(val);
      const answerCandidates = callDoc.collection("answerCandidates");
      const offerCandidates = callDoc.collection("offerCandidates");

      this.pc.onicecandidate = (event) => {
        event.candidate && answerCandidates.add(event.candidate.toJSON());
      };

      const callData = (await callDoc.get()).data();

      const offerDescription = callData.offer;
      await this.pc.setRemoteDescription(
        new RTCSessionDescription(offerDescription)
      );

      const answerDescription = await this.pc.createAnswer();
      await this.pc.setLocalDescription(answerDescription);

      const answer = {
        type: answerDescription.type,
        sdp: answerDescription.sdp,
      };

      await callDoc.update({ answer });

      offerCandidates.onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
          if (change.type === "added") {
            let data = change.doc.data();
            this.pc.addIceCandidate(new RTCIceCandidate(data));
          }
        });
      });
    }
  }
}
</script>
