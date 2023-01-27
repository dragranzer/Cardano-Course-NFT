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
- Simpan path dari policy kedalam variable
```
script="policy/policy.script"
```
- Buat keypair
```
cardano-cli address key-gen \
    --verification-key-file policy/policy.vkey \
    --signing-key-file policy/policy.skey
```
- Isi policy.script dengan menggunakan
```
echo "{" >> policy/policy.script
echo "  \"type\": \"all\"," >> policy/policy.script 
echo "  \"scripts\":" >> policy/policy.script 
echo "  [" >> policy/policy.script 
echo "   {" >> policy/policy.script 
echo "     \"type\": \"before\"," >> policy/policy.script 
echo "     \"slot\": $(expr $(cardano-cli query tip $testnet | jq .slot?) + 10000)" >> policy/policy.script
echo "   }," >> policy/policy.script 
echo "   {" >> policy/policy.script
echo "     \"type\": \"sig\"," >> policy/policy.script 
echo "     \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file policy/policy.vkey)\"" >> policy/policy.script 
echo "   }" >> policy/policy.script
echo "  ]" >> policy/policy.script 
echo "}" >> policy/policy.script
```
Sehingga menghasilkan:  
![image](https://user-images.githubusercontent.com/71221969/215033789-57fb42d3-bd95-4504-94a6-0051355f9ba6.png)
