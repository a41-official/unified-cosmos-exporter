# unified-cosmos-exporter

- 쿠버네티스에서 작동할 수 있는 exporter로서 log로 확인 가능
- 불필요 메트릭 삭제조치
```
{"time":"2025-09-04T02:07:42.895523178Z","level":"INFO","msg":"Analyzing missed block","height":10460412}
{"time":"2025-09-04T02:07:42.89813851Z","level":"INFO","msg":"New block detected","height":10460412,"proposer":"1A8E7A1590EF7B70B27C18DB9BF1C6E83C5F318D","chain_id":"zetachain_7000-1"}
{"time":"2025-09-04T02:07:42.898181519Z","level":"INFO","msg":"Block signatures","height":10460412,"total_signatures":75}
{"time":"2025-09-04T02:07:42.898191839Z","level":"INFO","msg":"Validator state","validator":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","consensus_address":"A8B80C641BD1940AFA336DC61DB69B050F4D8F33","active":true,"jailed":false}
{"time":"2025-09-04T02:07:42.898202719Z","level":"INFO","msg":"Checking validator signature","valoper":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","consensus_address":"A8B80C641BD1940AFA336DC61DB69B050F4D8F33"}
{"time":"2025-09-04T02:07:42.898212369Z","level":"INFO","msg":"Validator signed block","validator":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","moniker":"A41","height":10460412,"signed_blocks":241,"window_signed":241,"window_missed":0}
{"time":"2025-09-04T02:07:42.900703972Z","level":"INFO","msg":"New block detected","height":10460413,"proposer":"C6A7BF28E3015D3F68EAC8D78F17103CE7EE49B9","chain_id":"zetachain_7000-1"}
{"time":"2025-09-04T02:07:42.900761881Z","level":"INFO","msg":"Block signatures","height":10460413,"total_signatures":75}
{"time":"2025-09-04T02:07:42.900772021Z","level":"INFO","msg":"Validator state","validator":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","consensus_address":"A8B80C641BD1940AFA336DC61DB69B050F4D8F33","active":true,"jailed":false}
{"time":"2025-09-04T02:07:42.900781621Z","level":"INFO","msg":"Checking validator signature","valoper":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","consensus_address":"A8B80C641BD1940AFA336DC61DB69B050F4D8F33"}
{"time":"2025-09-04T02:07:42.900793351Z","level":"INFO","msg":"Validator signed block","validator":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","moniker":"A41","height":10460413,"signed_blocks":242,"window_signed":242,"window_missed":0}
{"time":"2025-09-04T02:07:42.900806911Z","level":"INFO","msg":"Validator state summary","validator":"zetavaloper1urqlulgyy8xda7fv83qp80ayzehl0px7hu3l3l","moniker":"A41","signed_blocks":242,"proposed_blocks":2,"missed_blocks":0,"consecutive_missed":0,"active":true,"jailed":false}
```

- 설치 방법

```
git clone https://github.com/steve-8000/unified-cosmos-exporter.git
````
```
cd unified-cosmos-exporter && go mod tidy && go build -o zeta-test-exporter
```
```
sudo docker build -t zeta-test-exporter .
```
```
sudo docker run -d --name zeta-test-exporter-metric -p 26300:26300 zeta-test-exporter
```


<config.yml 수정하기>
```
listen_address: ":26300"  # 수정 : exporter port 정보
metrics_interval: 30
logging:
  level: "debug"
  format: "json"

block_tracking:
  enabled: true
  interval: 5
  max_consecutive_missed: 100

prometheus:
  server: "http://<server ip>:26660" # 수정 : 벨리데이터 서버 ip:port 정보

chains:
  - chain_id: "xxx"  # 수정 : 체인 id
    name: "XXX Network"   # 수정 : 체인 이름
    rpc: "http://<server ip>:26657"  # 수정 : 벨리데이터 서버 ip:port 정보  
    api: "http://<server ip>:26617"  # 수정 : 벨리데이터 서버 ip:port 정보
    websocket: "ws://<server ip>:26657/websocket"  # 수정 : 벨리데이터 서버 ip:port 정보
    
    auto_detect: true
    
    validators:
      - "valoper"  # 수정 : 벨리데이터 주소값
    
    wallets:
      - address: "wallet address" # 수정 : 벨리데이터 월렛 주소
        name: "Main Wallet"
    
    peers:
      - "http://<server ip>:26657" # 수정 : 서버 ip 값(외부 ip 가능)
```
<Dockerfile 수정하기>
```
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o unified-exporter .

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/unified-exporter .
COPY config.yml ./
EXPOSE 9300 <이부분만 exporter 포트로 변경하기>
ENTRYPOINT ["./unified-exporter"]
```
```
curl http://localhost:26300/metrics
```