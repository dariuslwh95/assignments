# pre-requisites
- kubernetes environment (minikube)
- customised values.yaml with nodeport, generate self-signed ssl params enabled (visible in commit)
- bitnami helm chart, only values.yaml is edited as a best practice
# installation steps via helm
```
cd q1-nginx-helm
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade --install --debug --atomic --timeout=3m -f values.yaml my-nginx bitnami/nginx -n nginx --create-namespace
```

# validation
## helm installation
```
[07-14 13:05:34] osboxes:~/assignments/q1-nginx-helm$ helm list -n nginx
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-nginx        nginx           2               2024-07-14 04:49:39.938377629 +0000 UTC deployed        nginx-18.1.4    1.27.0     
```
## relevant resources
```
[07-14 13:05:52] osboxes:~/assignments/q1-nginx-helm$ kubectl get pod,deploy,service -n nginx
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-6df7647cd6-2sx47   1/1     Running   0          37m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   1/1     1            1           37m

NAME               TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/my-nginx   NodePort   10.107.134.111   <none>        80:30080/TCP,443:30081/TCP   37m
```
## ssl
```
[07-14 13:07:05] osboxes:~/assignments/q1-nginx-helm$ kubectl get nodes -owide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    <none>   26h   v1.28.3   192.168.49.2   <none>        Ubuntu 22.04.3 LTS   6.5.0-44-generic   docker://24.0.7
```
```
[07-14 13:07:22] osboxes:~/assignments/q1-nginx-helm$ openssl s_client -connect 192.168.49.2:30081
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN = my-nginx
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 CN = my-nginx
verify error:num=21:unable to verify the first certificate
verify return:1
depth=0 CN = my-nginx
verify return:1
---
Certificate chain
 0 s:CN = my-nginx
   i:CN = nginx-ca
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Jul 14 04:28:26 2024 GMT; NotAfter: Jul 14 04:28:26 2025 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
subject=CN = my-nginx
issuer=CN = nginx-ca
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1438 bytes and written 373 bytes
Verification error: unable to verify the first certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 2048 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 21 (unable to verify the first certificate)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: CB0310EF2A2D34F7C91E4886E0D4844A3FEC306D8301DC405AE8811611D84F5C
    Session-ID-ctx: 
    Resumption PSK: 2C34FFE1526BD060E61E2519B7AE1FFAF4E6013A992168F06D40E510B10B5DBD428EA3A599599845F15E12AA60855240
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 57 55 21 e9 1b 2b 87 7c-3a 64 33 6e d3 9c db 36   WU!..+.|:d3n...6
    0010 - 4f 62 7f 11 2e 53 50 f5-fd a9 44 b6 6e b8 ed b3   Ob...SP...D.n...
    0020 - 3d 1c 8a 11 99 b8 4d 24-c1 c8 6e 22 3a d4 4d 0e   =.....M$..n":.M.
    0030 - 6a 7d 61 70 98 81 30 a8-13 17 ff b0 02 11 e0 08   j}ap..0.........
    0040 - 8d 4d 09 3c 5a 49 93 b4-ee f8 95 2f 4e 21 72 a5   .M.<ZI...../N!r.
    0050 - 4f 7a bd 17 4f 9a cd 31-2f db 52 6b fe 60 44 c5   Oz..O..1/.Rk.`D.
    0060 - 3a ad 2e 34 12 9a d3 6f-bf de 18 82 85 3c 48 68   :..4...o.....<Hh
    0070 - 80 a0 22 d5 b7 c5 0b c7-52 40 14 48 96 2d 89 05   ..".....R@.H.-..
    0080 - 5f 0b 3b 78 b3 df a4 2a-8e 77 3b 3e d0 7c 45 5a   _.;x...*.w;>.|EZ
    0090 - e8 dd 72 45 c9 39 e8 e6-ba 8b f9 5f aa 79 37 2c   ..rE.9....._.y7,
    00a0 - c9 ab d2 6d 50 91 42 ce-3e c5 ac 3e 32 1d 6e 26   ...mP.B.>..>2.n&
    00b0 - 28 40 a4 cf e2 6a 5e 30-b4 fa 86 c6 ee 91 6d 6c   (@...j^0......ml
    00c0 - 11 68 91 6e ed 33 9a bb-a9 a7 44 57 fa f1 33 9d   .h.n.3....DW..3.
    00d0 - b2 71 f9 cf af cc 42 10-01 62 57 ab ac 12 58 bc   .q....B..bW...X.

    Start Time: 1720933663
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 745541F481DBCC163797F2CA83C982ECEFA196529A450C2E3D4CE0BCF7F2A2A2
    Session-ID-ctx: 
    Resumption PSK: C9FF455DECC8C37D3D728A9030CF359B16D39460826762C7B700C4D1A7878A50980E35DCC72CE40B3A05220B26F90B91
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 57 55 21 e9 1b 2b 87 7c-3a 64 33 6e d3 9c db 36   WU!..+.|:d3n...6
    0010 - fe 3c e3 d1 8d e8 b4 5f-0a 32 90 e8 45 b3 ed 06   .<....._.2..E...
    0020 - 0d 49 57 0d 6c 05 0d 15-4d 13 d0 45 ab 19 36 12   .IW.l...M..E..6.
    0030 - 4d 39 35 46 16 b9 0f 4d-60 8d 17 d6 ec 0b d8 52   M95F...M`......R
    0040 - 84 ea b4 a0 36 13 3a 5f-cb e5 b7 c1 43 e5 2d 49   ....6.:_....C.-I
    0050 - 4c 6f 7a 32 fa 22 f7 49-e7 b9 bc 80 ef 44 26 bc   Loz2.".I.....D&.
    0060 - 03 d4 b9 ca 66 e1 0d a2-52 7a 8f 4a 4b 67 f2 bb   ....f...Rz.JKg..
    0070 - 12 f5 25 0c 29 15 91 98-07 25 82 18 f4 5c 0f 84   ..%.)....%...\..
    0080 - b6 08 cc 6b 2a ad 90 ea-5d 9c 74 d4 46 9e b3 f7   ...k*...].t.F...
    0090 - 19 8c cc 14 f6 e8 aa 22-f4 b5 6c 7e 00 f1 2d 42   ......."..l~..-B
    00a0 - 0c 4b 49 13 9f dc 5a 63-33 ca c1 98 0c 53 35 91   .KI...Zc3....S5.
    00b0 - 5d 6a 34 25 38 4a ab 03-cf 43 fd 67 27 a8 3c 98   ]j4%8J...C.g'.<.
    00c0 - 3f 6f 5e 72 7d d0 df 15-e6 f9 0b 6d 90 f7 cb a2   ?o^r}......m....
    00d0 - 8b e4 40 6a ed fd 0e 33-15 95 79 95 7c 6c 59 55   ..@j...3..y.|lYU

    Start Time: 1720933663
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
    Extended master secret: no
    Max Early Data: 0
---
```