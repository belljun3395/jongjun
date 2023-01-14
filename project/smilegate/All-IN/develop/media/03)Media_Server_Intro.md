## Media Server - Intro

### Streaming protocol

Media Server에 앞서 스트리밍 프로토콜에 대해서 알아보자.

비디오 스트리밍 프로토콜은 기기 사이에서 비디오 또는 오디오의 내용을 담은 chunks를 보낸다.

이 chunks를 사용자의 장치에서 볼 수 있도록 하는 것을 reassembling이라고 한다.

즉, 사용자의 기기에서는 제공자가 사용한 프로토콜을 지원하여야 스트리밍을 할 수 있는 것이다.



#### RTMP

(출처 : https://blog.pogrebnyak.info/what-is-rtmp-and-how-its-used-in-live-streaming/)

우리가 이번 프로젝트에서 사용하는 프로토콜을 RTMP to HLS이다.

RTMP는 Real-Time Messaging Protocol의 비디오 스트리밍을 위한 적은 지연시간을 가진 프로토콜이다.



이 프로토콜의 장단점은 아래와 같다. 

##### ADVANTAGES 👍

- Based on TCP, that means **no lost frames and video glitches.**
- **Ultra low-latency (<1-2 second)** on good network conditions
- The de-facto **standard for streaming to CDNs and streaming platforms**
- Supports Encryption (RTPMS)

##### DISADVANTAGES 👎

- Based on TCP, that means higher **delay once transmission reach maximum bandwidth, or when network congestion occurs**
- No modern codecs support except **H264/AAC**
- Obsolete for playback

위에서 조금 더 살펴보면 좋을 부분을 강조해보았다.

장점은 TCP 기반으로 손실이 업고 적은 지연 시간을 가져 많은 CDN과 스트리밍 플렛폼의 표준이라는 점이다. 

( // TCP 기반이 손실이 없는 이유는 추후 공부후 보충 할 것이다.)

하지만 단점으로는 H263/ACC를 제외한 다른 모던 코덱을 지원하지 않는다는 것이다.

<img width="1161" alt="RTMP 소개" src="https://user-images.githubusercontent.com/102807742/212460852-b9aa062a-db1a-4940-bb00-36ec55f2ffb1.png">

사진은 nginx-rtmp 모듈로 만든 데모 미디어 서버인데 위의 사진에서도 코덱이 H263/ACC인 것을 확인할 수 있다.

또 네트워크 밴드 스위치가 최대가 되거나 네트워크 혼잡이 일어난다면 속도가 느려진다는 단점이 있다.

(// 이를 해결하면 성능을 개선 시킬 수 있다는 이야기가 될 것 같은데 이 부분에 대해서도 추후 공부해보아야겠다.)



##### RTMP URL

**`rtmp://host:port/app/stream-key`**는 스트리밍 송출을 위한 주소이다.

이때 stream-key는 방송 송출자를 구분할 수 있는 **유일 한 키이다.**



##### RTMP CHUNK STREAM

![rtmp-chunk-stream](https://blog.pogrebnyak.info/what-is-rtmp-and-how-its-used-in-live-streaming/rtmp-chunk-stream.jpg)

만약 우리가 1Mb 비디오 패킷을 보내려 하면 우리는 그것이 다 전송되기 전에는 다른 것을 전송하지 못할 것이다.

그렇기에 이 크기를 줄일 필요가 있고 1Mb 비디오 패킷을 일정 크기로 자른 것을 chunk라고 한다.

이 chunk는 기본이 128 bytes이고 50 Kb와 같이 그 크기를 조절 할 수도 있다.



##### RTMP STREAMING MESSAGES FLOW

```
Client >> << Server

>> TCP Connection Establish <<
>> RTMP Handshake <<

>> COMMAND_AMF0 ("connect", tx1, ...) >>
<< WINDOW_ACKNOWLEDEMENT_SIZE         <<
<< SET_PEER_BANDWIDTH                 <<
<< COMMAND_AMF0 ("_result", tx1, ...) <<

>> COMMAND_AMF0 ("createStream", tx2,...) >>
<< COMMAND_AMF0 ("_result", tx2, ...)     <<
 
>> COMMAND_AMF0 ("publish", tx3, ... streamId, live, ...)            >>
<< COMMAND_AMF0 ("onStatus", tx3, ...code="NetStream.Publish.Start") <<

>> DATA_AMF0 ("@setDataFrame", "onMetaData", ...) >>
>> VIDEO (AVCDecoderConfigurationRecord)          >>
>> AUDIO (AAC Sequence header)                    >>
   ...
>> VIDEO  >>
>> AUDIO  >>
   ...
>> COMMAND_AMF0 ("deleteStream")   >>

>> TCP Connection Drop <<
```

전체적인 클라이언트와 서버간의 메시지 플로우는 위와 같다고 한다.

위와 보이는 과정처럼 클라이언트와 서버간에는 많은 상호작용이 일어난다.

하지만 이를 전부 이해하지는 못하여 위의 설명도 부족하였는데 이부분은 추후 보충해보아야겠다.



### Media Server

먼길을 돌아 다시 미디어 서버이다.

우선 정의부터 살펴보면 다음과 같다. (출처 : https://en.wikipedia.org/wiki/Media_server)

---

A media server is a computer appliance or an application software that stores digital media (video, audio or images) and **makes it available over a network.**

---

비디오, 오디오, 이미지등을 네트워크를 통해 전달하는 것이 미디어 서버라 생각하면 될 것같다.

우선 우리는 비디오, 오디오를 네트워크를 통해 전달할 것이고 전달하는 방법은 RTMP to HLS이다.

이에 맞는 미디어서버를 찾아보았고 이때 고려 사항은 유료가 아니며 단순한 사용이 아닌 어느 정도 구현할 수 있는 것이어야 했다.

선택 기준에 성능에 대한 이야기가 없어 의아할 수 있다.

성능을 선택 기준으로 잡지 않은 것은 좋은 미디어 서버를 만드는 것이 우리 프로젝트의 목표는 아니고 성능이 좋은 미디어 서버는 유료 미디어 서버를 사용하면 되는 것이기 때문이다.

우리가 프로젝트의 목표는 대량의 트래픽을 받아보는 경험을 함으로써 이 경험을 통해 고민하고 공부하는 것을 통해 **개발자로써 성장**하는 것이 목표이기 때문이다.

성능이 좋은 미디어 서버를 찾는다면 "대량의 트래픽을 받아보는 경험"은 할 수 있겠지만 "개발자로써 성장"에는 도움이 되지 않을 것이라 생각하였다.

그래서 주로 찾아보았던 것은 미디어서버 오픈 소스이며 최종적인 선택은 Nginx with nginx-rtmp-module의 미디어 서버이다.

이는 기본 Nginx에 rtmp-module을 추가한 것이다.

Nginx 기반이기에 관련 자료와 레퍼런스가 충분할 뿐만 아니라 프로젝트에 필요한 검증과 같은 요구사항이 있다면 이를 Nginx를 기반으로 구현할 수도 있다는 장점이 있었다.



### Nginx with rtmp module Start

(출처 : https://qteveryday.tistory.com/371)

우선 Ubuntu 64-bit Arm 20.04.5를 기반으로 하며 nginx와 libnignx-mod-rtmp를 설치해야한다.

```she
// install nginx
sudo apt-get update

sudo apt-get upgrade -y

sudo apt-get install -y nginx 

// install libnignx-mod-rtmp
sudo apt update

sudo apt install -y libnginx-mod-rtmp
```

+ /etc/nginx : 기본 설정이 저장된 루트 디렉터리이다.
+ /etc/nginx/nginx.conf : 모든 설정에 대한 진입점이다. 최상위 http 블록을 가지고 있다.
+ /etc/nginx/conf.d/ : .conf로 끝나는 파일은 /etc/nginx/nginx.conf 파일이 가진 최사위 http 블록에 포함된다.



/etc/nginx/nginx.conf 기본 설정은 다음과 같다.

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}


http {
        # Basic Settings
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        # SSL Settings

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        server {
            listen       8080;
            server_name  localhost;

            # rtmp stat
            location /stat {
                rtmp_stat all;
                rtmp_stat_stylesheet stat.xsl;
            }

            location /stat.xsl {
                # you can move stat.xsl to a different location
                root /usr/build/nginx-rtmp-module;
            }

            # rtmp control
            location /control {
                rtmp_control all;
            }

            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }

        # Logging Settings

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        # Gzip Settings

        gzip on;


        # Virtual Host Configs

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```



http 블록만 선언되어 있는데 rtmp 프로토콜을 사용하려면 아래의 설정을 추가해주어야 한다.

```
# 영상을 미디어 서버로 보낼때 사용하는 프로토콜
# nginx rtmp 설정
rtmp {
        server {
                # rtmp 포트 번호
                listen 1935;
                listen [::]:1935 ipv6only=on; # 어디에서 오던 받아주도록 설정
                # allow_publish, deny_publish를 통해 특정 ip에서만 방송할 수 있도록 설정 가능하다.

                # rtmp 가 4k 블록으로 데이터 전송
                chunk_size 4096;

                application live {
                        live on;
                        record off;
                 }

        }
}
```



HLS로 변환을 하려면 `application live` 아래 다음 설정을 추가하면 된다.

```
#HLS
hls on;
hls_path /var/www/html/stream/hls; # hls 파일이 저장되는 위치
hls_fragment 3; 
hls_playlist_length 5;
```



위의 설정을 추가하여 완성한 rtmp 블록은 다음과 같다.

```
# 영상을 미디어 서버로 보낼때 사용하는 프로토콜
# nginx rtmp 설정
rtmp {
        server {
								# rtmp 포트 번호
                listen 1935;
		
								# rtmp 가 4k 블록으로 데이터 전송
                chunk_size 4096;
		
								# 동일한 서버에만 비디오 게시
                allow publish 127.0.0.1;
								allow publish all;
								#deny publish all;

		
								#  HLS 형식으로 변환 (트랜스먹싱 혹은 패킷타이징이라고 함)		
								# 스트림이 기본적으로 디스크에 저장안되게 처리
                application live {
                        live on;
                        record off;
               		
												#HLS
												hls on;
                        hls_path /var/www/html/stream/hls;
                        hls_fragment 3;
                        hls_playlist_length 60;

                        dash on;
                        dash_path /var/www/html/stream/dash;
		 						}
        }
}
```



이렇게 설정하고난 이후에는 rtmp가 사용하는 1935 포트 방화벽을 열고 nginx를 재시작 하여야 한다.

```
sudo ufw allow 1935/tcp
sudo systemctl reload nginx.service
```



그럼 이제 송출한 방송을 볼 수 있는 주소를 설정해 주어야 한다.

이는 /etc/nginx/sites-available/default 에서 설정하는데 이는 /etc/nginx/nginx.conf 에 include되어 있어 최상위 블럭은 http라 생각하면 된다.

이에 대한 설정은 다음과 같다.

```
# 미디어 서버
# rtmp -> hls 포트
server {
    listen 8088 ssl;

    ssl_certificate  # crt 파일 위치
    ssl_certificate_key # key 파일 위치

     location / {
        add_header Cache-Control no-cache;
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Expose-Headers' 'Content-Length';

			# 도메인:8088/ 의 root 경로임	
			root /var/www/html/stream; # /var/www/html/stream을 만들어 주어야 한다.

			types {
    		application/vnd.apple.mpegurl m3u8;
    	}
       
			# alias /tmp/hls/;
    }
}
```

위와 같이 설정한 이후 역시 8088포트를 열어주어야 한다.

```
sudo ufw allow 8088/tcp
```



이렇게 설정한 다음에는 `https:도메인:8088/hls/stream_key ` 에서 확인할 수 있다.

location 블록을 보면 '/' 이 선언되어 있다.

그리고 root로 '/var/www/html/stream' 이 설정되어 있다.

이는 '/' 이후 주소를 통해 '/var/www/html/stream' 이하 파일에 접근 할 수 있다는 의미다. 

그렇기에 '/var/www/html/stream' 이하 파일에서 볼려는 파일을 찾아야한다.

RTMP를 HLS로 바꾼 파일은 ' hls_path /var/www/html/stream/hls' 과 같이 설정하여 우선 **'/hls'**로 접근해야하고 

파일명이 stream_key로 저장되기 되어 **'/stream_key'** 접근하면된다.



이때 ssl 설정이 필요한데 local에서 이를 설정하는 방법은 [링크](https://imagineer.in/blog/https-on-localhost-with-nginx/)로 대체한다.

그리고 이 글을 찾기 위해 참고한 자료는 다음과 같다.

+ https://en.wikipedia.org/wiki/List_of_RTMP_software
+ https://antmedia.io/what-is-rtmp-server-how-to-set-up-a-free-rtmp-server/
+ https://blog.pogrebnyak.info/what-is-rtmp-and-how-its-used-in-live-streaming/
+ https://engineering.linecorp.com/ko/blog/the-structure-of-the-line-live-s-encoder-layer/#cdn-origin-serve
+ https://github.com/arut/nginx-rtmp-module
+ http://nginx-rtmp.blogspot.com/



Intro에서는 프로토콜, Nginx 그리고 RTMP to HLS를 간단히 구현해보았고 이후에 **스트리밍 전에 검증하는 방법** 그리고 **화질을 다양하게 하는 방법**을 공부한 이후 작성할 것이다.