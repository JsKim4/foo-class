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