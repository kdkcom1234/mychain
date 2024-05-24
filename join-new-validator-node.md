### 새로운 노드 추가 절차 (스테이킹 포함)

#### 1. 노드 초기화 및 계정 생성

새로운 노드에서 초기화하고 키를 생성합니다.

**새로운 노드:**

```sh
mychaind init validator-3 --chain-id mychain-devnet
mychaind keys add validator-3 --keyring-backend file
```

#### 2. 기존 블록체인 데이터 동기화

기존 노드에서 새로운 노드로 블록체인 데이터를 동기화합니다. 이를 위해 기존 노드의 `config.toml` 파일에서 `persistent_peers` 설정을 이용합니다.

**새로운 노드의 `config.toml` 파일 (`~/.mychaind/config/config.toml`):**

```toml
persistent_peers = "<node1_id>@<first_node_ip>:26656,<node2_id>@<second_node_ip>:26656"
```

새로운 노드에서 데이터를 동기화하려면 다음 명령어를 사용하여 노드를 시작합니다.

**새로운 노드:**

```sh
mychaind start
```

이 명령어는 기존 노드에서 블록체인 데이터를 가져와 동기화합니다.

#### 3. 노드 실행 및 동기화 확인

새로운 노드가 기존 노드와 동기화가 완료될 때까지 기다립니다. 동기화 상태를 확인하려면 로그를 모니터링하거나 REST API를 사용할 수 있습니다.

**로그 모니터링:**

```sh
tail -f ~/.mychaind/logs/mychaind.log
```

**REST API 사용:**

```sh
curl http://localhost:26657/status
```

#### 4. 새 계정에 코인 전송

새 계정을 생성하고 코인을 전송합니다. 이는 기존 노드에서 트랜잭션을 통해 이루어집니다.

**기존 노드:**

```sh
mychaind keys add new-account --keyring-backend file
mychaind tx bank send validator-1 <new-account_address> 1000000umy --chain-id mychain-devnet --keyring-backend file
```

여기서 `<new-account_address>`는 새로 생성한 계정의 주소입니다.

#### 5. 스테이킹을 통해 검증자로 설정

새로운 노드를 검증자로 설정하려면 스테이킹을 통해 검증자로 등록해야 합니다.

**새로운 노드:**

```sh
mychaind tx staking create-validator \
  --amount=5000000umy \
  --pubkey=$(mychaind tendermint show-validator) \
  --moniker="validator-3" \
  --chain-id=mychain-devnet \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=validator-3 \
  --keyring-backend=file
```

#### 6. 노드 실행 및 모니터링

새로운 노드가 네트워크에 정상적으로 참여하고 있는지 확인합니다. 블록이 정상적으로 생성되고 있는지, 트랜잭션이 정상적으로 처리되는지 모니터링합니다.

### 전체 과정 요약

1. 노드 초기화 (`mychaind init validator-3 --chain-id mychain-devnet`)
2. 키 추가 (`mychaind keys add validator-3 --keyring-backend file`)
3. 기존 블록체인 데이터 동기화 (기존 노드와 `persistent_peers` 설정)
4. 노드 실행 (`mychaind start`)
5. 동기화 상태 확인 (로그 모니터링 또는 REST API 사용)
6. 새 계정 생성 및 코인 전송 (`mychaind keys add new-account` 및 `mychaind tx bank send validator-1 <new-account_address> 1000000umy --chain-id mychain-devnet --keyring-backend file`)
7. 스테이킹을 통해 검증자로 설정 (`mychaind tx staking create-validator`)
8. 노드 실행 및 모니터링 (`mychaind start`)

이 과정을 통해 새로운 노드를 기존 네트워크에 추가하고 스테이킹을 통해 검증자로 설정할 수 있습니다.
