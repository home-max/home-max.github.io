<style type="text/css">
  @import url("/css/style-header.css");
</style>

# [맥쓰네 블로그](/ "https://max-jayee.github.io")

# Podman 설치 방법
## 설명
본 설치 방법은 공식 사이트에서 제공하는 버전을 기준으로 작성하였습니다.

## 설치 방법 // TODO: 설치 방법 정리
**설치 과정**
1. 기본 개발 도구 설치
2. 방화벽 개방
3. Gitlab 설치 (community edition(무료), enterprise edition(유료) 두가지 버전이 존재)
4. Gitlab 초기 설정

### CentOS 8 
1. podman 설치
```bash
sudo dnf install podman
```

2. image 다운로드
```bash
podman pull alpine:3.16.3
podman images
podman save > alpine-3.16.3.tar alpine:3.16.3
# or
podman save -o alpine-3.16.3.tar alpine:3.16.3
```

3. image 다운로드
```bash
podman load -i alpine-3.16.3.tar
podman load < alpine-3.16.3.tar
```

4. base docker file 작성 # TODO: alpine 내부에서 java 가 실행이 안됨
```bash
FROM alpine:3.16.3

ENV TZ=Asia/Seoul
ENV LANG=en_US.UTF-8

COPY OpenJDK8U-jdk_x64_linux_8u332b09.tar.gz /
RUN tar zxf /OpenJDK8U-jdk_x64_linux_8u332b09.tar.gz -C /usr/lib
#RUN rm /OpenJDK8U-jdk_x64_linux_8u342b07.tar.gz

ENV JAVA_HOME=/usr/lib/openjdk-8u332-b09
ENV PATH=${JAVA_HOME}/bin:${PATH}

ENV JAVA_VERSION=8u332

RUN chown root:root ${JAVA_HOME} -R
RUN chmod 755 ${JAVA_HOME} -R

#RUN java -version

CMD ["/bin/ash"]
```

5. base image 생성
```bash
podman build -t openjdk-1.8.0:0.0.1 ./
```

6. container 실행
```bash
podman run -itd --name base openjdk-1.8.0:0.0.1
podman ps
```

7. container 접근
```bash
podman exec -it base ash
```

8. nexus 에 저장 (only ip)
```bash
# nexushost: verser, port, 
podman tag image:version nexushost/repo/name:version

podman login nexushost

podman push nexushost/repo/name:version

#Getting image source signatures
#Checking if image destination supports signatures
#Error: Can not copy signatures to docker://localhost:5000/rhosp-rhel8/openstack-etcd:16.2: pinging container registry localhost:5000: Get "https://localhost:5000/v2/": http: server gave HTTP response to HTTPS client 발생시
/etc/containers/registries.conf.d/myregistry.conf
[[registry]]
location = "registry.mycluster.williamlieurance.com:5000"
insecure = true

[[registry]]
location = "localhost:5000"
insecure = true
```