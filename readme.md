## Install Go

```shell
wget go1.22.3.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
```

```shell
vi .profile
```

```
PATH=$PATH:/usr/local/go/bin
PATH=$PATH:$HOME/go/bin
```

## Install Ignite

```shell
curl https://get.ignite.com/cli! | bash
```

## Serve Development Mode

```shell
ignite chain serve -v
```

## Host Status

```shell
https://호스트명:26657/status
```

--

## Set Up Genesis

알겠습니다. 소스 코드 저장소와 제네시스 관련 데이터를 공유하는 저장소를 분리하여 전체 과정을 다시 작성하겠습니다.

### 전체 과정 요약

#### 0. 소스 코드 받기

먼저, 소스 코드가 있는 Git 저장소에서 코드를 클론합니다.

```sh
git clone <source-code-repo-url>
cd <source-code-repo-directory>
```

#### 1. 실행 파일 생성

`ignite`를 사용하여 실행 파일을 만듭니다.

```sh
ignite chain build
```

#### 2. 노드 초기화 및 계정 생성

각 노드에서 초기화하고 키를 생성합니다.

**첫 번째 노드:**

```sh
mychaind init validator-1 --chain-id mychain-devnet
mychaind keys add validator-1 --keyring-backend file
```

**두 번째 노드:**

```sh
mychaind init validator-2 --chain-id mychain-devnet
mychaind keys add validator-2 --keyring-backend file
```

#### 3. 키 주소 추출 및 공유

각 노드에서 생성된 키의 주소를 추출합니다.

**첫 번째 노드:**

```sh
mychaind keys show validator-1 -a --keyring-backend file
```

**두 번째 노드:**

```sh
mychaind keys show validator-2 -a --keyring-backend file
```

각 노드의 주소를 기록해 둡니다. 예를 들어:

- validator-1: my1...
- validator-2: my1...

두 번째 노드는 키 주소를 제네시스 관련 데이터를 공유하는 Git 저장소를 통해 첫 번째 노드에게 전달합니다.

**두 번째 노드:**

1. 키 주소를 제네시스 저장소에 추가하고 커밋합니다.

```sh
cd <genesis-repo-root>
echo "<validator-2_address>" > validator-2-address.txt
git add validator-2-address.txt
git commit -m "Add validator-2 address"
git push origin main
```

2. 첫 번째 노드에서 제네시스 저장소에서 변경 사항을 풀합니다.

**첫 번째 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
validator_2_address=$(cat validator-2-address.txt)
```

#### 4. 제네시스 파일 수정

첫 번째 노드에서 제네시스 파일을 수정하여 두 노드의 초기 잔액을 할당합니다.

**첫 번째 노드:**

```sh
mychaind genesis add-genesis-account validator-1 20000000umy --keyring-backend file
mychaind genesis add-genesis-account $validator_2_address 20000000umy --keyring-backend file
```

#### 5. 수정된 제네시스 파일 공유

수정된 제네시스 파일을 제네시스 저장소를 통해 공유합니다.

**첫 번째 노드:**

1. 수정된 `genesis.json` 파일을 제네시스 저장소에 추가하고 커밋합니다.

```sh
cd <genesis-repo-root>
cp ~/.mychaind/config/genesis.json ./genesis.json
git add genesis.json
git commit -m "Add modified genesis.json"
git push origin main
```

2. 두 번째 노드에서 제네시스 저장소에서 변경 사항을 풀합니다.

**두 번째 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
cp ./genesis.json ~/.mychaind/config/genesis.json
```

#### 6. 제네시스 파일 확인

두 번째 노드에서 제네시스 파일이 제대로 수정되었는지 확인합니다.

**두 번째 노드:**

```sh
cat ~/.mychaind/config/genesis.json
```

`genesis.json` 파일에서 `accounts` 섹션을 확인하여 두 계정이 올바르게 추가되었는지 확인합니다.

#### 7. `gentx` 생성

각 노드에서 `gentx`를 생성합니다.

**첫 번째 노드:**

```sh
mychaind genesis gentx validator-1 10000000umy --chain-id mychain-devnet --keyring-backend file
```

**두 번째 노드:**

```sh
mychaind genesis gentx validator-2 10000000umy --chain-id mychain-devnet --keyring-backend file
```

#### 8. `gentx` 파일 공유

두 번째 노드에서 생성된 `gentx` 파일을 제네시스 저장소를 통해 공유합니다.

**두 번째 노드에서 제네시스 저장소에 `gentx` 파일을 추가하고 커밋합니다:**

```sh
cd <genesis-repo-root>
cp ~/.mychaind/config/gentx/gentx-*.json ./gentx-2.json
git add gentx-2.json
git commit -m "Add gentx for validator-2"
git push origin main
```

첫 번째 노드는 두 번째 노드의 `gentx` 파일을 가져옵니다:

**첫 번째 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
cp ./gentx-2.json ~/.mychaind/config/gentx/
```

#### 9. `collect-gentxs` 실행

첫 번째 노드에서 `gentx` 파일을 모아서 `collect-gentxs`를 실행합니다.

**첫 번째 노드:**

```sh
mychaind genesis collect-gentxs --keyring-backend file
```

이 명령어는 `gentx` 파일을 모아서 제네시스 파일을 업데이트합니다.

#### 10. 제네시스 파일 수정 (denom 변경)

`collect-gentxs` 후에 `genesis.json` 파일에서 모든 `stake` denom을 `umy`로 변경합니다.

**첫 번째 노드:**

```sh
sed -i 's/"stake"/"umy"/g' ~/.mychaind/config/genesis.json
```

#### 11. 최신 제네시스 파일 공유

업데이트된 제네시스 파일을 제네시스 저장소를 통해 모든 노드에 공유합니다.

**첫 번째 노드:**

1. 수정된 `genesis.json` 파일을 제네시스 저장소에 추가하고 커밋합니다.

```sh
cd <genesis-repo-root>
cp ~/.mychaind/config/genesis.json ./genesis.json
git add genesis.json
git commit -m "Update genesis.json with new validators"
git push origin main
```

2. 모든 노드에서 제네시스 저장소에서 변경 사항을 풀합니다.

**모든 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
cp ./genesis.json ~/.mychaind/config/genesis.json
```

#### 12. Node ID 확인 및 공유

각 노드의 Node ID를 확인하고 제네시스 저장소를 통해 공유합니다.

**첫 번째 노드:**

```sh
node1_id=$(mychaind comet show-node-id)
echo $node1_id > node1-id.txt
git add node1-id.txt
git commit -m "Add Node ID for validator-1"
git push origin main
```

**두 번째 노드:**

```sh
node2_id=$(mychaind comet show-node-id)
echo $node2_id > node2-id.txt
git add node2-id.txt
git commit -m "Add Node ID for validator-2"
git push origin main
```

각 노드는 상대 노드의 Node ID를 가져옵니다:

**첫 번째 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
node2_id=$(cat node2-id.txt)
```

**두 번째 노드:**

```sh
cd <genesis-repo-root>
git pull origin main
node1_id=$(cat node1-id.txt)
```

#### 13. 설정 파일 편집

각 노드의 설정 파일을 편집합니다 (`~/.mychaind/config/config.toml` 및 `~/.mychaind/config/app.toml`).

##### 첫 번째 노드의 RPC 설정 변경

첫 번째 노드에서 RPC가 외부에서 접속할 수 있도록 `config.toml` 파일을 수정합니다.

**첫 번째 노드의 설정 파일 (`~/.mychaind/config/config.toml`):**

```toml
[rpc]
laddr = "tcp://0.0.0.0:26657"
```

##### Persistent Peers 설정

각 노드가 서로 연결될 수 있도록 `persistent_peers`를 설정합니다.

**첫 번째 노드의 설정 파일 (`~/.mychaind/config/config.toml`):**

```toml
persistent_peers = "$node2_id@<second_node_ip>:26656"
```

**두 번째 노드의 설정 파일 (`~/.mychaind/config/config.toml`):**

```toml
persistent_peers = "$node1_id@<first_node_ip>:26656"
```

##### 최소 가스 가격 설정

`app.toml` 파일에서 최소 가스 가격을 설정합니다.

**각 노드의 `app.toml` 파일 (`~/.mychaind/config/app.toml`):**

```toml
minimum-gas-prices = "1umy"
```

#### 14. 노드 실행

모든 노드에서 다음 명령어를 실행하여 노드를 시작합니다.

```

sh
mychaind start
```

#### 15. 테스트 및 모니터링

네트워크가 정상적으로 동작하는지 확인합니다. 블록이 정상적으로 생성되고 있는지, 트랜잭션이 정상적으로 처리되는지 모니터링합니다.

### 전체 과정 요약 (업데이트된 검증자 설정 및 RPC 설정)

0. 소스 코드 받기 (`git clone <source-code-repo-url>`)
1. 실행 파일 생성 (`ignite chain build`)
2. 노드 초기화 (`mychaind init <moniker> --chain-id mychain-devnet`)
3. 키 추가 (`mychaind keys add <validator_name> --keyring-backend file`)
4. 각 노드의 키 주소 추출 및 공유
   - 두 번째 노드에서 키 주소를 제네시스 저장소를 통해 첫 번째 노드에게 전달
5. 제네시스 파일 수정 (`mychaind genesis add-genesis-account <validator_address> 20000000umy --keyring-backend file`)
6. 수정된 제네시스 파일 공유 (제네시스 저장소를 통해 `genesis.json` 공유)
7. 제네시스 파일 확인 (`cat ~/.mychaind/config/genesis.json`)
8. `gentx` 생성 (`mychaind genesis gentx <validator_name> 10000000umy --chain-id mychain-devnet --keyring-backend file`)
9. `gentx` 파일 공유 (두 번째 노드에서 제네시스 저장소를 통해 `gentx` 파일 공유)
10. `collect-gentxs` 실행 (`mychaind genesis collect-gentxs --keyring-backend file`)
11. 제네시스 파일 수정 (denom 변경, `sed -i 's/"stake"/"umy"/g' ~/.mychaind/config/genesis.json`)
12. 최신 제네시스 파일 공유 (제네시스 저장소를 통해 `genesis.json` 파일 공유)
13. Node ID 확인 및 공유 (제네시스 저장소를 통해 Node ID 공유)
14. 설정 파일 편집 (`persistent_peers` 및 `minimum-gas-prices` 설정)
    - 첫 번째 노드의 RPC 설정 변경 (`laddr = "tcp://0.0.0.0:26657"`)
15. 노드 실행 (`mychaind start`)
16. 테스트 및 모니터링

이 과정을 통해 각 노드의 설정을 완료하고, 노드들이 서로 연결되며, 네트워크가 정상적으로 동작하도록 설정할 수 있습니다.

--
