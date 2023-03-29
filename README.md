# Hướng dẫn chạy node Base

## Thông tin dự án

- Website: https://base.org/
- Documentation: https://docs.base.org/
- Blog: https://base.mirror.xyz/
- Twitter: https://twitter.com/buildonbase
- Github: https://github.com/base-org
- Coinbase Faucet: https://coinbase.com/faucets/base-ethereum-goerli-faucet
- Quicknode Faucet: https://faucet.quicknode.com/drip
- Zora Mint: http://mint.base.org/
- Base x Optimism Mint: https://base.mirror.xyz/H_KPwV31M7OJT-THUnU7wYjOF16Sy7aWvaEr5cgHi8I
- Base Status: https://status.base.org/

## Yêu cầu VPS

- Tối thiểu 16gb RAM
- Ổ cứng SSD tối thiểu 100 Gb dung lượng trống
- Hệ điều hành Ubuntu > 18.04
- Các cổng (ports) cần mở (nếu vps có cấu hình firewall):
    + Layer1 (Geth):
        - 8545
        - 8546
        - 30303 (Bắt buộc)
    + Layer2 (Base Node):
        - 7545
        - 9222
        - 7300
        - 6060


## Chuẩn bị
*Lưu ý*: Có 2 RPC cần đăng ký



1. RPC của mạng **Base Goerli**
Vì mặc định sẽ dùng `https://goerli.base.org` nhưng vì giới hạn request nên đăng ký RPC riêng chạy cho khỏe

Chọn 1 trong các nhà cung cấp sau để đăng ký mạng testnet Base Goerli:
- [Quicknode](https://www.quicknode.com/chains/base)
- [Blockdaemon](https://try.blockdaemon.com/base-testnet/)
- [Infura](https://pages.consensys.net/infura-base-waitlist)

Sau khi đăng ký và có được địac chỉ RPC ví dụ (https://withered-empty-daylight.base-goerli.discover.quiknode.pro/a32a458404182c35c77523853f977461e24cc9/) **Gọi là RPC2 nhé**


2. RPC của mạng **ETH Goerli**
Mặc định đang là `https://ethereum-goerli-rpc.allthatnode.com` có thể đăng ký ở mấy nhà cung cấp trên nhé
Còn lười kiếm RPC public cũng được (Dùng thằng này sync node hơi chậm)
https://chainlist.org/?testnets=true&search=goerli

Ví dụ mình chọn thằng này: `https://goerli.blockpi.network/v1/rpc/public`
**Gọi là RPC1 nhé**

## Cài đặt

#### Kết nối đến VPS

```bash
ssh root@dia_chi_ip
```

#### Cài đặt docker
Chú ý nếu bạn `ssh` với user là `root` thì có thể bỏ `sudo` ở mỗi đầu dòng lệnh

```bash
sudo apt-get update -y
sudo apt-get install -y \
ca-certificates \
curl \
gnupg \
git \
snapd \
lsb-release

sudo snap install jq

sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

#### Cài đặt base node

- Tải code của dự án về:

```bash
git clone https://github.com/base-org/node.git
```
- `cd` vào thư mục node:

```bash
cd node
```
- Mở file `docker-compose.yml`
Dùng `nano` hoặc `vi` để edit file. Ở đây mình dùng nano thì dùng lệnh sau:

```
nano docker-compose.yml
```
#### Thay RPC1
Tìm tơi dòng `OP_NODE_L1_ETH_RPC` cập nhật bằng RPC mà bạn đăng ký ở trên (mục chuẩn bị)
Ví dụ:
Mặc định

```
- OP_NODE_L1_ETH_RPC=https://ethereum-goerli-rpc.allthatnode.com
```

Sửa lại:

```
- OP_NODE_L1_ETH_RPC=https://goerli.blockpi.network/v1/rpc/public
```

#### Thay RPC2
Tìm `OP_GETH_SEQUENCER_HTTP` và thay bằng RPC2 ví dụ:
```
- OP_GETH_SEQUENCER_HTTP=https://goerli.base.org
```
thay bằng
```
- OP_GETH_SEQUENCER_HTTP=https://withered-empty-daylight.base-goerli.discover.quiknode.pro/a32a458404182c35c77523853f977461e24cc9/
```

Xong bạn ấn tổ hợp phím `Ctrl + X` và ấn `y` để lưu lại

### Giờ là chạy node và kiểm tra

1. Chạy lệnh sau để chạy node

```
docker compose up -d
```

*Lưu ý*: Bạn phải đang ở trong thư mục node nhé. Nếu con trỏ đang không ở node thì cần `cd` vào node. Thường sẽ là `cd /root/node` nếu bạn login với `root`, còn user bình thường sẽ là `cd /home/tên/node`

2. Kiểm tra xem có docker chạy ổn chưa bằng lệnh:

```
docker ps
```

Để ý mục `Status` nếu nó ghi là Up ... mà ko có chữ restart đi kèm là ok nhé

3. Kiểm tra xem đã chạy chưa
Chạy lệnh sau:

```
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" http://localhost:8545
```

Nếu trả về kết quả như sau là được:

```
{"jsonrpc":"2.0","id":0,"result":{"baseFeePerGas":"0x3b9aca00","difficulty":"0x0","extraData":"0x424544524f434b","gasLimit":"0x17d7840","gasUsed":"0x0","hash":"0xa3ab140f15ea7f7443a4702da64c10314eb04d488e72974e02e2d728096b4f76","logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","miner":"0x4200000000000000000000000000000000000011","mixHash":"0x0000000000000000000000000000000000000000000000000000000000000000","nonce":"0x0000000000000000","number":"0x0","parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000","receiptsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","sha3Uncles":"0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347","size":"0x209","stateRoot":"0xacf9545e969af5b8aec09a78dd7f430f8aa318bc2d54f9e7a47c2ed3aefadff4","timestamp":"0x63d96d10","totalDifficulty":"0x0","transactions":[],"transactionsRoot":"0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421","uncles":[]}}
```

*Lưu ý*: Quá trình sync node có thể mất 1 tới 2 ngày

Kiểm tra quá trình sync

```
echo Latest synced block behind by: $((($(date +%s)-$( \  curl -d '{"id":0,"jsonrpc":"2.0","method":"optimism_syncStatus"}' \
  -H "Content-Type: application/json" http://localhost:7545 | \
  jq -r .result.unsafe_l2.timestamp))/60)) minutes
```

nó trả về thông tin kiểu

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2111    0  2056  100    55   4599    123 --:--:-- --:--:-- --:--:--  4733
Latest synced block behind by: 81793 minutes
```

4. Xem log

Muốn xem chi tiết chạy tới đâu thì có thể view log
```
docker-compose logs -f node
```

Xong rồi! Have fun!

À nhớ đăng ký nhé!


