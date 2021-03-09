==================== local ==================== 
 
    npm install -g artillery
    
테스트 코드 작성

    config:
      target: "http://{host}"
      phases:
        - duration: {테스트 시간}
          arrivalRate: {초당 요청}
          name: Warm up
    scenarios:
      # We define one scenario:
      - name: "just get hash"
        flow:
          - get:
              url: "/hash/123"
              

테스트 시작

    artillery.cmd run --output report.json .\cpu-test.yaml
    
테스트 결과 문서화

    artillery.cmd report .\report.json
    
실행중인 도커 컨테이너 조회

    docker ps
    
도커 종료

    docker conatiner kill -s 15 {docker-container-name}