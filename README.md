# Cardano-Course-NFT
Untuk membuat dan mengirim NFT dilakukan beberapa langkah langkah:
## 1. Membuat Variable pendukung
```
testnet="--testnet-magic 1"
realtokenname="IVAN"
tokenname=$(echo -n $realtokenname | xxd -b -ps -c 80 | tr -d '\n')
tokenamount="1"
fee="0"
output="2000000"
ipfs_hash="QmQNDKqzC3c6YB5yrXdPTLmLZBAzGiEBxWrdKJanJkEdqt"
```
- `output` adalah TADA yang di masukkan kedalam NFT
- `ipfs_hash` didapat dari CID setelah melakukan upload image ke `app.pinata`

## 2. Export protocol parameter
```
cardano-cli query protocol-parameters $testnet --out-file protocol.json
```
## 3. Buat Policy Script & PolicyID
- Buat folder dari policy dan file policy.script
```
mkdir policy
touch policy/policy.script && echo "" > policy/policy.script
```
