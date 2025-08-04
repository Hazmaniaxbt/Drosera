# Drosera Node Kurulumu ve Rol Alma Rehberi

Selamlar, bu repoda Hoodi Ethereum test ağında çalışan Drosera Trap ve Drosera Node kurulum adımlarını, ayrıca Discord’da Cadet rolünün nasıl alınacağını anlatıyoruz.

## Sistem Gereksinimleri:

- Ubuntu/Linux 22.04
- Minimum: 2 vCPU 4 RAM (benim kullandığım)
- Önerilen: 4vCPU 8 RAM
- Hoodi testnet'indeki test cüzdanı ve  Ethereum (private) kodu
  

## Hoodi Testnet ETH (Hoodi Token)
Faucetten testnet Eth alalım

- Minning yaparak alınabilir

  https://hoodi-faucet.pk910.de

  veya

  [QuickNode Faucet](https://faucet.quicknode.com/ethereum/hoodi)  
  [Stakely Faucet](https://stakely.io/faucet/ethereum-hoodi-testnet-eth)

  ## <h1 align="center">Kurulum</h1>
  
Kodları sırayla giriyoruz.

```bash
sudo apt-get update && sudo apt-get upgrade -y

sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```


## Docker kurulumu

```bash
sudo apt update -y && sudo apt upgrade -y

for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update

sudo apt-get install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Sonraki 4 satırı tek seferde kopyala yapıştır

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Tek tek girmeye devam

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo docker run hello-world
```

## 1. Drosera Trap Yükleme

### Gerekli Araçlar

```bash
cd
curl -L https://app.drosera.io/install | bash
source ~/.bashrc
droseraup
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

###  Trap Projesini Başlatma

```bash
mkdir ~/my-drosera-trap
cd ~/my-drosera-trap

git config --global user.email "github_emailinizi_yazınız"   # Tırnakları kaldırmadan içine yazın
git config --global user.name "github_kullanıcıadını_yazınız"   # Tırnakları kaldırmadan içine yazın

forge init -t drosera-network/trap-foundry-template
```

### Trap Build

```bash
bun install
forge build
```

### Trap Konfigurasyon ayarı

```bash
cd ~/my-drosera-trap
nano drosera.toml
```
drosera.toml dosyanının içine aşağıdan "Testnet_cüzdan_adresi" düzenleyin yapıştırın. 
Sonra CTRL + X sonra Y ve enterlayıp çıkın

### Trap Konfigurasyon (`drosera.toml`)

```toml
ethereum_rpc = "https://rpc.hoodi.ethpandaops.io/"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.helloworld]
path = "out/HelloWorldTrap.sol/HelloWorldTrap.json"
response_contract = "0x183D78491555cb69B68d2354F7373cc2632508C7"
response_function = "helloworld(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["Testnet_cüzdan_adresi"]  # Tırnakları kaldırmadan içine yazın

```

### Apply Uygulayın

eth_private_key yerine testnet cüzdanınızın private kodunu yapıştırın
  
```bash
DROSERA_PRIVATE_KEY=eth_private_key drosera apply
```
### Dashboard da trapınızı kontrol edin

- [https://app.drosera.io/](https://app.drosera.io/)\   
- Dashboarda gidin
- EVM Cüzdanınızı bağlayın
- Hoodi testnet ağına geçtiğinizden emin olun.
- Arama çubuğuna cüzdan adrsinizi yapıştırıp aratın

![telegram-cloud-photo-size-4-5798519704791403886-y](https://github.com/user-attachments/assets/b33f62e9-c9e9-4746-a5e1-88bf32352ef5)

 "Send Bloom Bost" ile bir miktar testEth gönderin. Faucetten token aldıkça yatırdığınız eth miktarını yükseltin. 
 
 Daha sonra operator kaydı yapacağız.


 # <h1 align="center">Drosera Operator Kurulumu</h1>

### Drosera Network klasörü oluşturma

```bash
cd
mkdir ~/Drosera-Network
cd ~/Drosera-Network
```
###  `docker-compose.yaml` düzenleme
```bash
nano docker-compose.yaml
```

### Açılan dosyaya (docker-compose.yaml) aşağıdaki içeriği olduğu gibi yapıştırın. Herhangi bir değişiklik yapmadan CTRL + X sonra Y ve enterleyip çıkın

```yaml
version: '3.8'

services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    ports:
      - "31313:31313"   # P2P
      - "31314:31314"   # HTTP
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://rpc.hoodi.ethpandaops.io/
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
      - RUST_LOG=info,drosera_operator=debug
      - DRO__ETH__RPC_TIMEOUT=30s
      - DRO__ETH__RETRY_COUNT=5
      # Optional: increase internal peer retry behavior (if supported by app)
      # - DRO__NETWORK__RETRY_INTERVAL=10s
      # - DRO__NETWORK__RETRY_COUNT=5
    volumes:
      - drosera_data:/data
    restart: always   # Ensures automatic container recovery
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
    healthcheck:   # Add Docker-native watchdog
      test: ["CMD", "curl", "-f", "http://localhost:31314/health"]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 30s
    command: node

volumes:
  drosera_data:
```

### .env dosyası oluşturma
Aynı klasörde  `nano .env` yapıp, env dosyası oluşturun. açılan dosyanın içine aşağıdaki iki satırı yapıştırın. 
- istenen yerleri düzenleyin. CTRL +X sonra Y ve enterlayıp çıkın

```env
ETH_PRIVATE_KEY=eth_private_key
VPS_IP=sunucu_ip_adresi
```
###  Docker Image yükleme
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

###  Operatorü Çalıştırma
```bash
docker compose up -d
```

```bash
screen -S drosera
docker compose logs -f
```

Logları kontrol edin "successful" aldıysanız tamamdır. Screenden çıkmak için CTRl + A + D

## B. Operator Kaydı
- Eth_private_key yazan yere testnet cüzdan private kodunuzu yapıştırın sonra enterlayın
- İşlemi ofc yazarak onaylayın.

```bash
drosera-operator register \
  --eth-rpc-url https://rpc.hoodi.ethpandaops.io \
  --eth-private-key eth_private_key \
  --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```
- Bu kodlardan sonra registered - succesful gibi ibareler görmelisiniz

## C. Trap Operator Kaydı
trap_address yazan yere dashboarddan trap config yazan yere tıklayarak alabilirsiniz ve bu adresi bir yete not alıp saklayın.

eth_private_key yazan yere testnet cüzdan private kodunuzu yapıştırın ve enterlayın.

![telegram-cloud-photo-size-4-5798519704791403885-y](https://github.com/user-attachments/assets/32e3f2cc-f0f1-41f2-900e-0d5b8ba9a100)

```bash
drosera-operator optin \
  --eth-rpc-url https://rpc.hoodi.ethpandaops.io \
  --eth-private-key eth_private_key_here \
  --trap-config-address trap_address
```
 - Sorarsa ofc yazarak onaylayın yoksa devam
 - Bu kodlardan sonra registered - succesful gibi ibareler görmelisiniz

## D. Port ayarlarını Yapın

```bash
sudo ufw enable -y
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
```

### Dashboarda Operator Kaydı
Opt-in zaten yapmıştık

![telegram-cloud-photo-size-4-5798519704791403886-y](https://github.com/user-attachments/assets/c2f99f7d-ec09-499f-87f8-1febb4adcc75)


Biraz bekleyince yeşiller görünmeye başladıysa operatorünüz başarılıyla kuruldu demektir. 

![telegram-cloud-photo-size-4-5798519704791403887-y](https://github.com/user-attachments/assets/da313400-77f3-458f-832c-c9e29202784b)


## <h1 align="center">Discord Rolü Alma</h1>
### 1-
```bash
cd ~/my-drosera-trap
```

#### 2- Yeni Trap.sol dosyası oluşturma

```bash
nano src/Trap.sol
```

#### 3- Aşağıdaki içeriği dosyanın içine yapıştırın istenen yeri değiştirin.
"YOURDISCORD" yerine discord kullanıcı adınızı yazın. İsminizi değil kullanıcı adınızı burası önemli (tırnakları kaldırmayın içine yazın)
CTRL +X sonra Y ve enterlayıp çıkın

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    // Updated response contract address
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "YOURDISCORD"; // Replace with your Discord username

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }

        return (true, abi.encode(name));
    }
}
```



---

### 2.  `drosera.toml` dosyasını düzenleme

```bash
nano drosera.toml
```

Dosyanın içini aşağıdaki ile değiştirip istenen yerleri girin
- OPERATOR_ADDRESS  = Testnet cüzdan adresi
- TRAP_CONFIG_ADDRESS = Trap adresiniz

Ctrl + X sonra Y ve enterlayıp çıkın

```toml
ethereum_rpc = "https://rpc.hoodi.ethpandaops.io"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.mytrap]
path = "out/Trap.sol/Trap.json"
response_contract = "0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"
response_function = "respondWithDiscordName(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["OPERATOR_ADDRESS"]
address = "TRAP_CONFIG_ADDRESS"
```

--

### 3. Trap çalıştıma

#### 1- Build etme
```bash
forge build
```

```bash
source /root/.bashrc
```


#### 2- Çalıştırma
```bash
drosera dryrun
```


### xxx yerine testnet cüzdan private kodunu yapıştırın

```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```

Eğer sorarsa çıkan ekrana ofc yazarak enterlayın yoksa devam

---

### 4. İşlemi kontrol etme
- OWNER_ADDRESS yerine testnet cüzdan adresinizi yapıştırın.
- sonuç "true" dönmesi gerek, görürseniz bu aşama tamamdır.

```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "isResponder(address)(bool)" OWNER_ADDRESS --rpc-url https://rpc.hoodi.ethpandaops.io
```


---

### 5. Operatorü tekrar çalıştırma

```bash
cd
cd ~/Drosera-Network
docker compose down -v
docker compose up -d
```
### 6. Listede Discord isminizi gördüyseniz tamamdır

```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "getDiscordNamesBatch(uint256,uint256)(string[])" 0 2000 --rpc-url https://rpc.hoodi.ethpandaops.io
```
---
![telegram-cloud-photo-size-4-5798519704791403888-y](https://github.com/user-attachments/assets/08c1de00-9dbe-47e5-a34b-30ce9c7bdb0c)

### Screen tekrar çalıştırıp loglarımıza bakalım. Screen'den çıkmak için CTRL + A + D

```bash
screen -r drosera
docker compose logs -f
```

