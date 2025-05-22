npm 설치 테스트 진행

<!-- LibreChat 시작 -->
docker compose up -d
docker compose up

<!-- 실행 중인 컨테이너를 중지 -->
docker compose down


<!-- 폴더 삭제 -->
# 1. 모든 컨테이너 중지 및 삭제
<!-- docker stop $(docker ps -aq) -->
<!-- docker rm $(docker ps -aq) -->

# 2. LibreChat 관련 이미지 삭제
<!-- docker images | grep librechat -->
<!-- docker rmi $(docker images | grep librechat | awk '{print $3}') -->

# 3. 사용되지 않는 네트워크/볼륨 정리 (선택)
<!-- docker network prune -f -->
<!-- docker volume prune -f -->

# 4. 시스템 전체 정리 (주의: 다른 프로젝트 영향 가능)
<!-- docker system prune -a --volumes -f -->



<!-- 뭔가 될것 같음 -->
cd LibreChat
docker compose up -d

docker ps
확인

cd client
npm run dev


<!-- 실행 중이거나 종료된 컨테이너 목록 확인 -->
docker ps -a

<!-- 모든 컨테이너 중지 -->
docker stop $(docker ps -aq)

<!-- 모든 컨테이너 삭제 -->
docker rm $(docker ps -aq)

<!-- 이미지 목록 확인 -->
docker images

<!--  -->
docker rmi $(docker images | grep librechat | awk '{print $3}')
docker rmi $(docker images -q) <!-- 이미지 전체 삭제 -->

<!-- 네트워크 목록 확인 -->
docker network ls

<!-- 사용되지 않는 네트워크 모두 삭제 (확인 메시지 뜸) -->
docker network prune

<!-- 볼륨 목록 확인 -->
docker volume ls

<!-- 사용되지 않는 볼륨 모두 삭제 -->
docker volume prune

<!-- Docker 시스템 전체 캐시 정리 -->
<!-- 주의: 다른 프로젝트에서 쓰는 것까지 삭제될 수 있으니 신중히 사용하세요. -->
docker system prune -a --volumes

<!-- LibreChat 폴더 삭제 -->
rm -rf ./LibreChat
