# Membuat koneksi simpel data canel WebRTC

1. Pada server penyedia canel

```
const lc = new RTCPeerConnection()
const dc = lc.createDataChannel("channel")

// akan otomatis dijalankan jika akan menerima pesan dari client
dc.onmessage = e => console.log("Pesan dari client yang join canel : "+e.data)

// akan otomatis dijalankan jika client berhasil terhubung ke client
dc.onopen = e => console.log("[server] Koneksi terbuka")

// membuat ice candidate untuk server ( rdp / remote description protocol )
lc.onicecandidate = e => console.log("[Server] Ice Candidate Baru "+JSON.stringify(lc.localDescription))

// membuat penawaran
// note : saat variabel lc local description di set dengan offer maka lc.onicecandidate akan otomatis aktif dan akan mecetak ice candidate (rdp protocol)
lc.createOffer().then(o => {
    return lc.setLocalDescription(o)
}).then(a => {
    console.log("Penawaran berhasil di buat")
})

```

output : `[Server] Ice Candidate Baru {"type":"offer","sdp":"v=0\r\no=- 5540519069971358003 3 IN IP4 127.0.0.1\r\ns=-\r\nt=0 0\r\na=group:BUNDLE 1\r\na=extmap-allow-mixed\r\na=msid-semantic: WMS\r\nm=application 54480 UDP/DTLS/SCTP webrtc-datachannel\r\nc=IN IP4 192.168.233.77\r\na=candidate:654563958 1 udp 2122260223 192.168.233.77 54480 typ host generation 0 network-id 1 network-cost 10\r\na=ice-ufrag:f24h\r\na=ice-pwd:OSf9xyvEU+0teQRoppMyWM34\r\na=ice-options:trickle\r\na=fingerprint:sha-256 4D:0D:29:70:3B:5C:4E:73:EA:E1:C3:4E:84:0D:43:5A:87:78:A7:4F:F1:3B:CF:8C:77:B7:4C:E7:29:FB:FC:6A\r\na=setup:actpass\r\na=mid:1\r\na=sctp-port:5000\r\na=max-message-size:262144\r\n"}`

2. Pada client join canel

```
// offer ini di inisial berdasarkan output dari ice candidate server
const offer = {"type":"offer","sdp":"v=0\r\no=- 5540519069971358003 3 IN IP4 127.0.0.1\r\ns=-\r\nt=0 0\r\na=group:BUNDLE 1\r\na=extmap-allow-mixed\r\na=msid-semantic: WMS\r\nm=application 54480 UDP/DTLS/SCTP webrtc-datachannel\r\nc=IN IP4 192.168.233.77\r\na=candidate:654563958 1 udp 2122260223 192.168.233.77 54480 typ host generation 0 network-id 1 network-cost 10\r\na=candidate:1770006150 1 tcp 1518280447 192.168.233.77 9 typ host tcptype active generation 0 network-id 1 network-cost 10\r\na=ice-ufrag:f24h\r\na=ice-pwd:OSf9xyvEU+0teQRoppMyWM34\r\na=ice-options:trickle\r\na=fingerprint:sha-256 4D:0D:29:70:3B:5C:4E:73:EA:E1:C3:4E:84:0D:43:5A:87:78:A7:4F:F1:3B:CF:8C:77:B7:4C:E7:29:FB:FC:6A\r\na=setup:actpass\r\na=mid:1\r\na=sctp-port:5000\r\na=max-message-size:262144\r\n"}

const rc = new RTCPeerConnection()

// membuat ice candidate untuk client ( rdp / remote description protocol )
rc.onicecandidate = e => console.log("[Client] Ice Candidate Baru "+JSON.stringify(rc.localDescription))

// function otomatis di panggil, jika nanti terdapat data canel pada pada permintaan server
rc.ondatachannel = e => {
    rc.dc = e.channel
    rc.dc.onmessage = e => console.log("pesan dari server yang buat canel "+e.data)
    rc.dc.onopen = e => console.log("[client] Koneksi Terbuka")
}

// set protokol tdp server pada client
rc.setRemoteDescription(offer).then(e => {
    console.log("[Berhasil] Set data offer dari server")
})

// client memberikan jawaban atas permintaan server
rc.createAnswer().then(a => {
    return rc.setLocalDescription(a)
}).then(a => {
    console.log("[Berhasil] Memeberikan jawaban dan membuat jawaban baru atas permintaan")
})

// ketika jawaban telah diberikan makan onicecandidate akan otomatis aktif dan memberikan protokol drp baru
```

output : `[Client] Ice Candidate Baru {"type":"answer","sdp":"v=0\r\no=- 5505246276171588980 2 IN IP4 127.0.0.1\r\ns=-\r\nt=0 0\r\na=group:BUNDLE 1\r\na=extmap-allow-mixed\r\na=msid-semantic: WMS\r\nm=application 56965 UDP/DTLS/SCTP webrtc-datachannel\r\nc=IN IP4 192.168.233.77\r\na=candidate:654563958 1 udp 2122260223 192.168.233.77 56965 typ host generation 0 network-id 1 network-cost 10\r\na=ice-ufrag:Cdhx\r\na=ice-pwd:I90YkEQFW+N7PEPb7NzVT7b/\r\na=ice-options:trickle\r\na=fingerprint:sha-256 ED:DB:7B:EA:76:E8:FD:DF:83:5A:FD:56:A9:99:73:58:B5:D9:FF:E8:91:5C:21:F2:E3:F1:73:08:08:45:D8:2A\r\na=setup:active\r\na=mid:1\r\na=sctp-port:5000\r\na=max-message-size:262144\r\n"}`

3. Pada server penyedia canel

```
// protokol rdp jawaban dari client atas permintaan server connect ke client
const aswer = {"type":"answer","sdp":"v=0\r\no=- 5505246276171588980 2 IN IP4 127.0.0.1\r\ns=-\r\nt=0 0\r\na=group:BUNDLE 1\r\na=extmap-allow-mixed\r\na=msid-semantic: WMS\r\nm=application 56965 UDP/DTLS/SCTP webrtc-datachannel\r\nc=IN IP4 192.168.233.77\r\na=candidate:654563958 1 udp 2122260223 192.168.233.77 56965 typ host generation 0 network-id 1 network-cost 10\r\na=ice-ufrag:Cdhx\r\na=ice-pwd:I90YkEQFW+N7PEPb7NzVT7b/\r\na=ice-options:trickle\r\na=fingerprint:sha-256 ED:DB:7B:EA:76:E8:FD:DF:83:5A:FD:56:A9:99:73:58:B5:D9:FF:E8:91:5C:21:F2:E3:F1:73:08:08:45:D8:2A\r\na=setup:active\r\na=mid:1\r\na=sctp-port:5000\r\na=max-message-size:262144\r\n"}

// server melakukan remote terhadap protokol rdp jawaban dari client untuk melakukan remote atau koneksi ke client
lc.setRemoteDescription(aswer)

```

# Mengirimkan pesan

1. Pada server penyedia canel

```
// untuk mengirim pesan ke client
dc.send("Hello ini dari server")
```

2. Pada client join canel

```
output#  pesan dari server yang buat canel : Hello ini dari server

// untuk mengirim pesan balik ke server
rc.dc.send("Hello ini dari client")
```

3. Pada server penyedia canel

```
output#  Pesan dari client yang join canel : Hello ini dari client
```
