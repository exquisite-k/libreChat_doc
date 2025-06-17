# **🛠️ 명령어 및 기타 작업 가이드(K)**

## **📁 LibreChat 디렉터리 구조**
```
LibreChat/
├── client/                         <- 프론트엔드 디렉터리 구조
│   ├── src/                        <- 프론트엔드 코드 (React 기반)
│   ├── public/                     <- 정적 리소스
│
├── api/                            <- 백엔드 디렉터리 구조
│   ├── server/                     <- 백엔드 API 서버
│
├── .env                            <- 환경 변수 파일
├── docker-compose.override.yml     <- Docker 설정 파일
├── librechat.yaml                  <- librechat 설정 파일
├── README.md                       <- 프로젝트 설명(markdown)
```

## **📝 Docker 실시간 로그 조회**
```bash
# 현재 로그 조회
docker compose logs

# 실시간 로그 조회
docker compose logs -f

# 디스크 사용량 조회
docker system df
```
> --- 
> * `docker compose logs -f` 명령어는 실시간으로 계속 노출되며, 종료는 따로 명령해주어야 합니다. (Ctrl + c)
> ---

---

## **🧱 Docker 컨테이너**
```bash
# 컨테이너 목록 조회
docker ps -a

# 전체 컨테이너 중지
docker stop $(docker ps -aq)

# 전체 컨테이너 시작
docker start $(docker ps -aq)

# 모든 컨테이너 삭제
docker rm $(docker ps -aq)
```

---

## **🖼️ Docker 이미지**
```bash
# 모든 이미지 목록 조회
docker images -a

# 모든 이미지 삭제
docker rmi $(docker images -aq)

# librechat(특정 프로젝트) 관련 최종 이미지 목록 조회
docker images | grep librechat 

# librechat(특정 프로젝트) 관련 최종 이미지 삭제
docker rmi $(docker images | grep librechat | awk '{print $3}')
```

---

## **🔊 Docker 볼륨**
```bash
# 볼륨 목록 조회
docker volume ls

# 사용되지 않는 볼륨 모두 삭제 (--force)
docker volume prune -f
```

---

## **💻 Docker 네트워크**
```bash
# 네트워크 목록 조회
docker network ls

# 사용되지 않는 네트워크 모두 삭제 (--force)
docker network prune -f
```

---

## **⏳ 초기화**

### **Docker 모든 시스템 전체 초기화**
*`[주의]다른 프로젝트에서 쓰는 것까지 삭제될 수 있으니 신중히 사용하세요.`*
```bash
docker system prune -a --volumes -f
```

### **볼륨 제외 초기화**

```bash
docker system prune -a -f
```

### **단계별 초기화**
```bash
docker container prune -f
docker image prune -a -f
docker network prune -f
docker volume prune -f
```
    
| 항목 | `docker system prune -a -f` | `docker system prune -a --volumes -f` |
| --- | --- | --- |
| 삭제 대상: **중지된 컨테이너** | 포함 | 포함 |
| 삭제 대상: **미사용 이미지** | 포함 (`-a` 옵션으로 중간 + 사용되지 않는 이미지 포함) | 포함 |
| 삭제 대상: **미사용 네트워크** | 포함 | 포함 |
| 삭제 대상: **미사용 볼륨** | `삭제 X` (볼륨은 기본적으로 제외됨) | `함께 삭제됨` (중요 데이터까지 삭제될 수 있음) |
| 데이터 삭제 위험성 | 이미지/컨테이너 위주 정리 | 볼륨 데이터까지 영구 삭제 (복구 불가) |
| 추천 상황 | 이미지/컨테이너만 정리하고 싶은 경우 | 전체 클린업이 필요할 때 (테스트 후 전체 초기화 등) |
| 주의사항 | 삭제 전 `docker system df` 확인 권장 | 삭제 전 `볼륨 데이터 백업 필수`, 중요 서비스에는 사용 금지 |

---

## **🗑️ LibreChat 폴더 삭제**
```bash
# 현재 메인 디렉터리가 LibreChat인 경우
rm -rf ../LibreChat
```
> ---
> * 현재 디렉터리 위치가 `LibreChat`이라면 `rm -rf ../LibreChat`을 사용합니다.
> * 현재 디렉터리 위치가 `LibreChat`이 아니라면 현재 디렉터리 위치를 변경하거나,
>   삭제할 폴더의 위치를 변경해주어야 합니다.
> ---