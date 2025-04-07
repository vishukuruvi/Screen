<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sender - Share Screen</title>
</head>
<body>
  <h2>📤 Screen Sender</h2>
  <button onclick="startScreenShare()">Start Sharing Screen</button>

  <h3>Offer (Send to Receiver)</h3>
  <textarea id="offer" rows="10" cols="60" readonly></textarea>

  <h3>Paste Answer (From Receiver)</h3>
  <textarea id="answer" rows="10" cols="60"></textarea>
  <button onclick="addAnswer()">Add Answer</button>

  <script>
    let peer = new RTCPeerConnection();

    async function startScreenShare() {
      const stream = await navigator.mediaDevices.getDisplayMedia({ video: true });
      stream.getTracks().forEach(track => peer.addTrack(track, stream));

      const offer = await peer.createOffer();
      await peer.setLocalDescription(offer);

      // Wait for ICE candidates to complete
      peer.onicegatheringstatechange = () => {
        if (peer.iceGatheringState === 'complete') {
          document.getElementById('offer').value = JSON.stringify(peer.localDescription);
        }
      };
    }

    async function addAnswer() {
      const answerText = document.getElementById('answer').value;
      if (!answerText) return alert("Please paste the receiver's answer.");
      const answer = JSON.parse(answerText);

      if (!peer.currentRemoteDescription) {
        await peer.setRemoteDescription(answer);
        console.log("Receiver's answer set!");
      } else {
        console.log("Remote description already set.");
      }
    }

    peer.onicecandidate = event => {
      if (event.candidate) {
        console.log("Sender ICE Candidate:", event.candidate);
      }
    };
  </script>
</body>
</html>
