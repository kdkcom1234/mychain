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
ignite chain serve -v --config ./config-validator-1.yml
ignite chain serve -v --config ./config-validator-2.yml
ignite chain serve -v --config ./config-validator-3.yml
```

## Host Status

```shell
https://호스트명:26657/status
```
