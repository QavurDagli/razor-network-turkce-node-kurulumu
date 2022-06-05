[![Coverage Status](https://coveralls.io/repos/github/razor-network/razor-go/badge.svg?branch=main)](https://coveralls.io/github/razor-network/razor-go?branch=main)

# Razor Network Türkçe Node Kurulumu 

Resmi github sayfasından forklanarak tarafımdan düzenlenmiştir.

# Sistem Gereksinimleri

4 CPU (Arm64 ya da Amd64)
4GB RAM

## Metamask'a Ağ Eklenmesi ve Fee için Token Alınması 

Öncelikle Skale Testnet v2 ağını metamask'a ekliyoruz.

Network Name:	Skale Testnet v2
New RPC URL:	https://staging-v2.skalenodes.com/v1/whispering-turais
Chain ID:	132333505628089
Currency Symbol:	ETH
Block Explorer URL:	https://whispering-turais.testnet-explorer.skalenodes.com

100.000 adet Razor tokenini cüzdanımızda görmek için metamaska aşağıdaki kontratı ekliyoruz;
Contract address: 0xDc342De801De342EdE013C7E6e7a78Fa4f548041

ETH fee için faucetten token alıyoruz: https://faucet.skale.network/
Skale Endpoint yazan yere bu adresi yapıştıyoruz: https://staging-v2.skalenodes.com/v1/whispering-turais

## Node Kurulumu

### Güncellemeler ve Docker Kurulumu
Sunucuya root yetkisi ile erişim sağladıktan sonra aşağıdaki kodları sırasıyla giriyoruz.
```
apt-get update
apt install docker.io
snap install docker
docker --version
docker run hello-world
docker images
```

### Go Kurulumu

```
wget -O go1.17.3.linux-amd64.tar.gz https://golang.org/dl/go1.17.3.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.3.linux-amd64.tar.gz && rm go1.17.3.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```
Son çıktı go versionun 1.17 olduğunu belirtiyorsa bu adım da tamamlanmış demektir.

### Razor Netwok Node Kurulumu
```
docker run -d -it --name razor-go -v "$(echo $HOME)"/.razor:/root/.razor razornetwork/razor-go:v1.0.3-incentivised-testnet-phase2-patch2
```
#### Yapılandırmayı ayarlıyoruz.
```
docker exec -it razor-go razor setConfig --provider https://staging-v2.skalenodes.com/v1/whispering-turais --gasmultiplier 1 --buffer 20 --wait 30 --gasprice 0 --logLevel debug --gasLimit 2
```

#### 100.000 Razor aldığımız cüzdanımızı import ediyoruz.
```
docker exec -it razor-go razor import
```
Kodu girdikten sonra karşımıza aşağıdaki gibi bir ekran gelecek;
```
$ ./razor import
🔑 Private Key:
Password:
```
Burada üzdanımızın private key'ini girip şifre belirliyoruz.

#### Stake işlemini gerçekleştiriyoruz.
```
docker exec -it razor-go razor addStake --address <cüzdan adresimizi yazıyoruz> --value <stake edeceğimizi miktar> --logFile logs
```
Aşağıdaki örnekte gibi olacak:
```
docker exec -it razor-go razor addStake --address 0x0000000000000000000000000000000000000000 --value 99000 --logFile logs
```

#### Delegasyonu Ayarlama
Node'unuzu delegation işlemlerine yani delegatorlerin size stake etmesine açmak için aşağıdaki kodları giriyoruz;
```
docker exec -it razor-go razor setDelegation --address 0x0000000000000000000000000000000000000000 --status true --commission 5 --logFile logs
```
comission bölümünü istediğiniz gibi değiştirebilirsiniz ben 5 yazdım.

#### Oylamalara Katılma
Oylamalalara katılmak ve işlemlerin arka planda gerçekleşmesi için için tmux yüklüyoruz;
```
apt install tmux
```
tmux'u çalıştırıyoruz
```
tmux new -s razor-go
```
Oylamaları yapacağımız cüzdan adresimizi giriyoruz;
```
docker exec -it razor-go razor vote --address 0x0000000000000000000000000000000000000000 --logFile logs
```

### Yararlı Komutlar

#### Komisyonu Güncelleme 

```
docker exec -it razor-go razor updateCommission --address <address> --commission <commission_percent>
```

Örneğin:

```
$ ./razor updateCommission --address 0x0000000000000000000000000000000000000000 --commission 5
```

### Delegate Etmek

Eğer bir validatore token stake etmek yani delege etmek isterseniz aşağıdaki kodu kullanabilirsiniz.
`<address>` bölümüne stake edeceğiniz adresi, `<value>`bölümüne stake edeceğiniz miktarı `<staker_id>` bölümüne  is provided, their stake is increased.



```
docker exec -it razor-go razor delegate --address <address> --value <value> --pow <power> --stakerId <staker_id>
```

Örneğin:

```
docker exec -it razor-go razor delegate --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --value 1000 --pow 10 --stakerId 1
```

#### Komisyonları Çekme 


```
docker exec -it razor-go razor claimCommission --address <address> 
```

Örneğin:

```
docker exec -it razor-go razor claimCommission --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c 
```

#### Oylama

Razor stake ederek oylamaya katılabilirsiniz.

```
docker exec -it razor-go razor vote --address <address>
```

Arka planda oylama komutunu çalıştırmak için aşağıdaki kodu da kullanabilirsiniz. Yukarıda yazmıştım.

```
docker exec -it -d razor-go razor vote --address <address> --password /root/.razor/<file_name>
```
>**_NOTE:_**  To run command with password flag, password file should present in $HOME/.razor/ directory


## Aşağıdaki Bölümleri Daha Sonra Düzenleyeceğim.

Example:

```
$ ./razor vote --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c
```
If you want to claim your bounty automatically after disputing staker, you can just pass `--autoClaimBounty` flag in your vote command.

If you want to report incorrect values, there is a `rogue` mode available. Just pass an extra flag `--rogue` to start voting in rogue mode and the client will report wrong medians.
The rogueMode key can be used to specify in which particular voting state (commit, reveal) or for which values i.e. medians/revealedIds (medians, missingIds, extraIds, unsortedIds)you want to report incorrect values.

Example:

```
$ ./razor vote --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --rogue --rogueMode commit,reveal,medians,missingIds,extraIds,unsortedIds
```

### Unstake

If you wish to unstake your funds, you can run the `unstake` command.

razor cli

```
$ ./razor unstake --address <address> --stakerId <staker_id> --value <value> --pow <power>
```

docker

```
docker exec -it razor-go razor unstake --address <address> --stakerId <staker_id> --value <value> --pow <power>
```

Example:

```
$ ./razor unstake --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --stakerId 1 --amount --pow 10 1000
```

### Withdraw

Once `unstake` has been called, you can withdraw your funds using the `initiateWithdraw` and `unlockWithdraw` commands

You need to start the withdrawal process using `initiateWithdraw` command and once the withdraw lock period is over you can use `unlockWithdraw` command to get the RZR's back to your account.

razor cli

```
$ ./razor initiateWithdraw --address <address> --stakerId <staker_id>
```

```
$ ./razor unlockWithdraw --address <address> --stakerId <staker_id>
```
docker

```
docker exec -it razor-go razor initiateWithdraw --address <address> --stakerId <staker_id>
```

```
docker exec -it razor-go razor unlockWithdraw --address <address> --stakerId <staker_id>
```

Example:

```
$ ./razor initiateWithdraw --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --stakerId 1
```
```
$ ./razor unlockWithdraw --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --stakerId 1
```

### Extend Lock

If the withdrawal period is over, then extendLock can be called to extend the lock period.

razor cli

```
$ ./razor extendLock --address <address> --stakerId <staker_id>
```

docker

```
docker exec -it razor-go razor extendLock --address <address> --stakerId <staker_id>
```

Example:

```
$ ./razor extendLock --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --stakerId 1
```

### Claim Bounty

If you want to claim your bounty after disputing a rogue staker, you can run `claimBounty` command

>**_NOTE:_**  bountyIds are stored in .razor directory with file name in format `YOUR_ADDRESS_disputeData.json file.` 
> 
> e.g: `0x2EDc3c6F93e4e20590F480272AB490D2620557xY_disputeData.json`

If you know the bountyId, you can pass the value to `bountyId` flag.

razor cli

```
$ ./razor claimBounty --address <address> --bountyId <bounty_id>
```

docker

```
docker exec -it razor-go razor claimBounty --address <address> --bountyId <bounty_id>
```

Example:

```
$ ./razor claimBounty --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --bountyId 5
```

You can also run claimBounty command without passing `bountyId` flag as it will pick up bountyIds associated to your address from the file one at a time.

razor cli

```
$ ./razor claimBounty --address <address> 
```

docker

```
docker exec -it razor-go razor claimBounty --address <address> 
```

### Transfer

Transfers razor to other accounts.

razor cli

```
$ ./razor transfer --value <value> --to <transfer_to_address> --from <transfer_from_address>
```

docker

```
docker exec -it razor-go razor transfer --value <value> --to <transfer_to_address> --from <transfer_from_address>
```

Example:

```
$ ./razor transfer --value 100 --to 0x91b1E6488307450f4c0442a1c35Bc314A505293e --from 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c
```

### Create Job

Create new jobs using `creteJob` command.

_Note: This command is restricted to "Admin Role"_

razor cli

```
$ ./razor createJob --url <URL> --selector <selector_in_json_or_XHTML_selector_format> --selectorType <0_for_XHTML_or_1_for_JSON> --name <name> --address <address> --power <power> --weight <weight>
```

docker

```
docker exec -it razor-go razor createJob --url <URL> --selector <selector_in_json_or_XHTML_selector_format> --selectorType <0_for_XHTML_or_1_for_JSON> --name <name> --address <address> --power <power> --weight <weight>
```

Example:

```
$ ./razor createJob --url https://www.alphavantage.co/query\?function\=GLOBAL_QUOTE\&symbol\=MSFT\&apikey\=demo --selector '[`Global Quote`][`05. price`]" --selectorType 1 --name msft --power 2 --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --weight 32
```

OR

```
$  ./razor createJob --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c -n btc_gecko --power 2 -s 'table tbody tr td span[data-coin-id="1"][data-target="price.price"] span' -u https://www.coingecko.com/en --selectorType 0 --weight 100
```

### Create Collection

Create new collections using `creteCollection` command.

_Note: This command is restricted to "Admin Role"_

razor cli

```
$ ./razor createCollection --name <collection_name> --address <address> --jobIds <list_of_job_ids> --aggregation <aggregation_method> --power <power> --tolerance <tolerance>
```

docker

```
docker exec -it razor-go razor createCollection --name <collection_name> --address <address> --jobIds <list_of_job_ids> --aggregation <aggregation_method> --power <power> --tolerance <tolerance>
```

Example:

```
$ ./razor createCollection --name btcCollectionMean --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --jobIds 1,2 --aggregation 2 --power 2 --tolerance 200
```

### Modify Collection Status

Modify the active status of an collection using the `modifyCollectionStatus` command.

_Note: This command is restricted to "Admin Role"_

razor cli

```
$ ./razor modifyCollectionStatus --collectionId <collectionId> --address <address> --status <true_or_false>
```

docker

```
docker exec -it razor-go razor modifyCollectionStatus --collectionId <collectionId> --address <address> --status <true_or_false>
```

Example:

```
$ ./razor modifyCollectionStatus --collectionId 1 --address 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --status false
```

### Update Collection

Update the collection using `updateCollection` command.

_Note: This command is restricted to "Admin Role"_

razor cli

```
$ ./razor updateCollection --collectionId <collection_id> --jobIds <list_of_jobs> --address <address> --aggregation <aggregation_method> --power <power> --tolerance <tolerance>
```

docker

```
docker exec -it razor-go razor updateCollection --collectionId <collection_id> --jobIds <list_of_jobs> --address <address> --aggregation <aggregation_method> --power <power> --tolerance <tolerance>
```

Example:

```
$ ./razor updateCollection -a 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --collectionId 3 --jobIds 1,3 --aggregation 2 --power 4 --tolerance 5
```

### Update Job

Update the existing parameters of the Job using `updateJob` command.

_Note: This command is restricted to "Admin Role"_

razor cli

```
./razor updateJob --address <address> --jobID <job_Id> -s <selector> --selectorType <selectorType> -u <job_url> --power <power> --weight <weight>
```

docker

```
docker exec -it razor-go razor updateJob --address <address> --jobID <job_Id> -s <selector> --selectorType <selectorType> -u <job_url> --power <power> --weight <weight>
```

Example:

```
$ ./razor updateJob -a 0x5a0b54d5dc17e0aadc383d2db43b0a0d3e029c4c --jobId 1 -s last -u https://api.gemini.com/v1/pubticker/btcusd --power 2 --weight 10
```

### Job details

Get the list of all jobs with the details like weight, power, Id etc.

Example:

razor cli

```
$ ./razor jobList
```

docker

```
docker exec -it razor-go razor jobList
```

### Collection details

Get the list of all collections with the details like power, Id, name etc.

Example:

razor cli

```
$ ./razor collectionList
```

docker

```
docker exec -it razor-go razorcollectionList
```

Note : _All the commands have an additional --password flag that you can provide with the file path from which password must be picked._



### Expose Metrics
Expose Prometheus-based metrics for monitoring

Example:

razor cli

```
$ ./razor setConfig --exposeMetrics 2112
```

docker

```
# Create docker network

docker network create razor_network

# Expose Metrics
docker exec -it razor-go razor setConfig --exposeMetrics 2112
```

### Override Job and Adding Your Custom Jobs

Jobs URLs are a placeholder from where to fetch values from. There is a chance that these URLs might either fail, or get razor nodes blacklisted, etc.
You can override the existing job and also add your custom jobs by adding `assets.json` file in `.razor` directory so that razor-nodes can fetch data directly from the provided jobs.

Shown below is an example of how your `assets.json` file should be -
```
{
  "assets": {
    "collection": {
      "ethCollectionMean": {
        "power": 2,
        "official jobs": {
          "1": {
            "URL": "https://data.messari.io/api/v1/assets/eth/metrics",
            "selector": "[`data`][`market_data`][`price_usd`]",
            "power": 2,
            "weight": 2
          },
        },
        "custom jobs": [
          {
            "URL": "https://api.lunarcrush.com/v2?data=assets&symbol=ETH",
            "selector": "[`data`][`0`][`price`]",
            "power": 3,
            "weight": 2
          },
        ]
      }
    }
  }
}
```

Breaking down into components
- The existing jobs that you want to override should be included in `official jobs` and fields like URL, selector should be replaced with your provided inputs respectively.

In the above example for the collection `ethCollectionMean`, job having `jobId:1` is override by provided URL, selector, power and weight.
```
"official jobs": {
          "1": {
            "URL": "https://data.messari.io/api/v1/assets/eth/metrics",
            "selector": "[`data`][`market_data`][`price_usd`]",
            "power": 2,
            "weight": 2
          },
```

- Additional jobs that you want to add to a collection should be added in `custom jobs` field with their respective URLs and selectors.

In the above example for the collection `ethCollectionMean`, new custom job having URL `https://api.lunarcrush.com/v2?data=assets&symbol=ETH` is added.
```
 "custom jobs": [
          {
            "URL": "https://api.lunarcrush.com/v2?data=assets&symbol=ETH",
            "selector": "[`data`][`0`][`price`]",
            "power": 3,
            "weight": 2
          },
        ]
```

### Logs

User can pass a separate flag --logFile followed with any name for log file along with command. The logs will be stored in ```.razor``` directory.

razor cli
```
$ ./razor addStake --address <address> --value <value> --logFile stakingLogs
```
docker
```
docker exec -it razor-go razo addStake --address <address> --value <value> --logFile stakingLogs
```
_The logs for above command will be stored at "home/.razor/stakingLogs.log" path_

razor cli
```
$ ./razor delegate --address <address> --value <value> --pow <power> --stakerId <staker_id> --logFile delegationLogs
```
docker
```
docker exec -it razor-go razo delegate --address <address> --value <value> --pow <power> --stakerId <staker_id> --logFile delegationLogs
```
_The logs for above command will be stored at "home/.razor/delegationLogs.log" path_

_Note: If the user runs multiple commands with the same log file name all the logs will be appended in the same log file._


### Contract Addresses

This command provides the list of contract addresses.

razor cli

```
$ ./razor contractAddresses
```

docker

```
docker exec -it razor-go razor contractAddresses
```

Example:

```
$ ./razor contractAddresses
```


### Setting up razor-go and commands using docker-compose

1. Must have `docker` and `docker-compose` installed
2. Building the source `docker-compose build`
3. Create razor.yaml at $HOME/.razor/

   ```bash
   vi $HOME/.razor/razor.yaml
   ```

4. Add in razor.yaml and use :wq to exit form editor

   ```bash
   buffer: 20
   gaslimit: 2
   gasmultiplier: 1
   gasprice: 0
   provider: <rpc-url>
   chainId: 80001
   wait: 30
   ```

5. Create account , and note address.

   ```bash
   docker-compose run razor-go /usr
   /local/bin/razor create
   ```

6. Import account

   ```
   docker-compose run razor-go /usr/local/bin/razor import
   ```

7. Get some **RAZOR** and **MATIC** token (or Token of respective RPC) to this address
8. Start **Staking**

   ```bash
   #Provide password through CLI
   docker-compose run razor-go /usr/local/bin/razor addStake --address <address> --value 50000

   #Provide password through File

     #Create file and put password string
       vi ~/.razor/pass
     #Start Staking
       docker-compose run razor-go /usr/local/bin/razor addStake --address <address> --value 50000 --password /root/.razor/pass

   ```

9. To Start **Voting**,

   1. Provide password through **CLI**

   ```bash
   # Run process in foreground and provide password through cli
   docker-compose run razor-go /usr/local/bin/razor vote --address <address>

   # Run process in background and provide password through file
   docker-compose run -d razor-go /usr/local/bin/razor vote --address <address> --password /root/.razor/pass
   ```

   1. Provide password through **File** and run in background with compose up
      1. replace <address> in docker-compose.yml with your address and create file pass and add your password in file

   ```bash
   docker-compose up -d
   ```

10. Enable Delegation

    ```bash
    #Provide password with cli
    docker-compose run razor-go /usr/local/bin/razor setDelegation --address <address> --status true --commission 10

    #provide password through file
    docker-compose run razor-go /usr/local/bin/razor setDelegation --address <address> --status true --commission 10 --password /root/.razor/pass
    ```

### Contribute to razor-go

We would really appreciate your contribution. To see our [contribution guideline](https://github.com/razor-network/razor-go/blob/main/.github/CONTRIBUTING.md)
