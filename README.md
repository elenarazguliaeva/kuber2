# kuber2
//////////////////////////////////////////////////////////////////
Smoke Test
/////////////////////////////////////////////////////////////////////

root@vm7:/home/elena.razgulayeva# kubectl create secret generic kubernetes-the-hard-way \
>   --from-literal="mykey=mydata"
secret/kubernetes-the-hard-way created
root@vm7:/home/elena.razgulayeva# gcloud compute ssh controller-0 \
>   --command "sudo ETCDCTL_API=3 etcdctl get \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem\
>   /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a c0 b0 60 bd e4 2e c8  |:v1:key1:..`....|
00000050  19 e3 94 36 9a 2a 46 89  e2 fe d2 50 ee e9 54 f6  |...6.*F....P..T.|
00000060  47 a8 10 07 b4 42 08 64  bc 33 35 97 7f 58 0e cd  |G....B.d.35..X..|
00000070  15 1f 3e e3 40 4e 36 ac  e7 ff 69 9b 1a 0c 63 37  |..>.@N6...i...c7|
00000080  9d fa 50 51 27 3e 2b b7  a8 77 ef f3 6e 1f 83 32  |..PQ'>+..w..n..2|
00000090  11 25 f7 13 d4 fa fa bc  95 fc 82 2a 5e a4 ad 42  |.%.........*^..B|
000000a0  a5 ab 27 9b e8 eb b4 c9  c4 c9 4a 5c f9 a8 1b eb  |..'.......J\....|
000000b0  b5 a8 a2 31 12 9b 82 af  e3 4c f5 f1 e7 f2 40 85  |...1.....L....@.|
000000c0  b2 f6 3f 01 6b 3d db cf  fc 8b 0a 15 72 50 c1 10  |..?.k=......rP..|
000000d0  83 1e 0f 54 4f 14 27 fa  69 c0 d9 46 10 e5 89 bd  |...TO.'.i..F....|
000000e0  7a 09 13 68 9c 38 51 3a  8c 25 5a 58 4f 85 2e eb  |z..h.8Q:.%ZXO...|
000000f0  c7 76 bd 95 e7 2e 31 33  ae 70 66 a5 7d b2 4f 4c  |.v....13.pf.}.OL|
00000100  cf c0 d4 3b ec d9 b1 a7  c4 23 2d 2f 49 f4 f9 ac  |...;.....#-/I...|
00000110  07 52 63 a1 69 1a 34 59  55 50 2f b9 e2 0f de c3  |.Rc.i.4YUP/.....|
00000120  1b ff ba 54 d5 9c 45 73  f9 f9 87 00 4c 8f a3 84  |...T..Es....L...|
00000130  31 32 ee 66 4d 25 54 80  86 ed 6f 83 15 69 32 cd  |12.fM%T...o..i2.|
00000140  6f e6 96 cf c6 5f f8 f4  7d 44 f8 2b d2 9e ca 31  |o...._..}D.+...1|
00000150  7b a1 a9 b5 12 dd 8f 0c  94 0a                    |{.........|
0000015a
root@vm7:/home/elena.razgulayeva# kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
root@vm7:/home/elena.razgulayeva# kubectl get pods -l app=nginx
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-xz5s8   1/1     Running   0          13s
root@vm7:/home/elena.razgulayeva# POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
root@vm7:/home/elena.razgulayeva# kubectl port-forward $POD_NAME 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^Croot@vm7:/home/elena.razgulayeva# kubectl logs $POD_NAME
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
127.0.0.1 - - [18/Mar/2021:12:20:58 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.68.0" "-"
root@vm7:/home/elena.razgulayeva# kubectl exec -ti $POD_NAME -- nginx -v
nginx version: nginx/1.19.8
root@vm7:/home/elena.razgulayeva# kubectl expose deployment nginx --port 80 --type NodePort
service/nginx exposed
root@vm7:/home/elena.razgulayeva# NODE_PORT=$(kubectl get svc nginx \
>   --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
root@vm7:/home/elena.razgulayeva# gcloud compute firewall-rules create kubernetes-the-hard-way-allow-nginx-service \
>   --allow=tcp:${NODE_PORT} \
>   --network kubernetes-the-hard-way
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/amazing-zephyr-304415/global/firewalls/kubernetes-the-hard-way-allow-nginx-service].
Creating firewall...done.
NAME                                         NETWORK                  DIRECTION  PRIORITY  ALLOW      DENY  DISABLED
kubernetes-the-hard-way-allow-nginx-service  kubernetes-the-hard-way  INGRESS    1000      tcp:30532        False
root@vm7:/home/elena.razgulayeva# EXTERNAL_IP=$(gcloud compute instances describe worker-0 \
>   --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')
root@vm7:/home/elena.razgulayeva# curl -I http://${EXTERNAL_IP}:${NODE_PORT}
HTTP/1.1 200 OK
Server: nginx/1.19.8
Date: Thu, 18 Mar 2021 12:24:01 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 09 Mar 2021 15:27:51 GMT
Connection: keep-alive
ETag: "604793f7-264"
Accept-Ranges: bytes

root@vm7:/home/elena.razgulayeva# echo EXTERNAL_IP
EXTERNAL_IP
root@vm7:/home/elena.razgulayeva# echo ${EXTERNAL_IP}
35.247.40.147
root@vm7:/home/elena.razgulayeva# echo ${NODE_PORT}
30532
root@vm7:/home/elena.razgulayeva#
