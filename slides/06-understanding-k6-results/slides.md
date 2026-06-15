# k6 결과

---

## 테스트 종료 요약 보고서

다시 한번 해당 출력을 확인해 봅시다:

```shell
$ k6 run test.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0001] Hello world!                                  source=console

running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU

     data_received..................: 5.9 kB 9.0 kB/s
     data_sent......................: 564 B  860 B/s
     http_req_blocked...............: avg=524.18ms min=524.18ms med=524.18ms max=524.18ms p(90)=524.18ms p(95)=524.18ms
     http_req_connecting............: avg=123.28ms min=123.28ms med=123.28ms max=123.28ms p(90)=123.28ms p(95)=123.28ms
     http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
       { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
     http_req_failed................: 0.00%  ✓ 0        ✗ 1
     http_req_receiving.............: avg=165µs    min=165µs    med=165µs    max=165µs    p(90)=165µs    p(95)=165µs
     http_req_sending...............: avg=80µs     min=80µs     med=80µs     max=80µs     p(90)=80µs     p(95)=80µs
     http_req_tls_handshaking.......: avg=399.48ms min=399.48ms med=399.48ms max=399.48ms p(90)=399.48ms p(95)=399.48ms
     http_req_waiting...............: avg=129.94ms min=129.94ms med=129.94ms max=129.94ms p(90)=129.94ms p(95)=129.94ms
     http_reqs......................: 1      1.525116/s
     iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
     iterations.....................: 1      1.525116/s

```

---

## k6 내장 메트릭

### 응답 시간

```shell
http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

---

### http_req_duration

> 💡 `http_req_duration`은 *모든* 요청에 대한 값입니다.

아래 줄은 *성공한 요청만*에 대한 응답 시간을 보고합니다.

```shell
  { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

---

### 오류율

`http_req_failed` 메트릭은 테스트의 오류율을 나타냅니다. 오류율은 전체 요청 중 테스트 중 실패한 요청의 비율입니다.

```shell
http_req_failed................: 0.00%  ✓ 0        ✗ 1
```

---

### 요청 수

테스트 중 모든 VU가 전송한 총 요청 수는 아래 줄에 표시됩니다.

```shell
http_reqs......................: 1      1.525116/s
```

---

### Iteration duration

iteration duration은 k6가 VU 코드의 단일 루프를 수행하는 데 걸린 시간입니다.

```shell
iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
```

---

### Iteration 수

iteration 수는 k6가 모든 VU의 iteration을 포함하여 스크립트를 총 몇 번 반복했는지를 나타냅니다.

```plain
iterations.....................: 1      1.525116/s
```

---

## k6 스크립트에 check 추가하기

- 이동: [07-adding-checks-to-your-script](?p=07-adding-checks-to-your-script)
