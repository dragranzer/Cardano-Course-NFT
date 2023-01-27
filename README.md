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
Sehingga `policy.script` menjadi:  
![image](https://user-images.githubusercontent.com/71221969/215033789-57fb42d3-bd95-4504-94a6-0051355f9ba6.png)
  
- Simpan variable `slotnumber`
```
slotnumber=19115858
```
- Generate PolicyID
```
cardano-cli transaction policyid --script-file ./policy/policy.script > policy/policyID
```
## 4. Buat Metadata
```
echo "{" >> metadata.json
echo "  \"721\": {" >> metadata.json 
echo "    \"$(cat policy/policyID)\": {" >> metadata.json 
echo "      \"$(echo $realtokenname)\": {" >> metadata.json
echo "        \"description\": \"Emurgo NFT lesson\"," >> metadata.json
echo "        \"name\": \"Emurgo NFT token\"," >> metadata.json
echo "        \"id\": \"1\"," >> metadata.json
echo "        \"image\": \"ipfs://$(echo $ipfs_hash)\"" >> metadata.json
echo "      }" >> metadata.json
echo "    }" >> metadata.json 
echo "  }" >> metadata.json 
echo "}" >> metadata.json
```
Sehingga `metadata.json` akan terlihat seperti:  
![image](https://user-images.githubusercontent.com/71221969/215037170-eff0a0fe-87a9-49d3-8c1c-5bb56c381cca.png)
  
## 5. Buat beberapa variable pembantu
```
txhash="50a2a3bcd1e7533307fd7fde69daaa03b26affef7f9090f77e41bb31267abdac"
txix="0"
funds="10000000000"
policyid=$(cat policy/policyID)
address=addr_test1qrqkckltx9khqtpqsdfszd96l24e352wuwwvj7nykt657vea4ufdupveqvpgkugwwf9uh34nq7nwn20hmrlmzvnrfx4s9qtjse
```
- `txhash` dan `txix` diambil dari wallet milik pembuat NFT  
- `address` adalah alamat dari wallet penerima
![image](https://user-images.githubusercontent.com/71221969/215037633-00a1c674-c217-4524-9f82-e7cb15a96c15.png)

## 6. Build Minting Transaksi
```
cardano-cli transaction build \
$testnet \
--babbage-era \
--tx-in $txhash#$txix \
--tx-out $address+$output+"$tokenamount $policyid.$tokenname" \
--change-address $address \
--mint="$tokenamount $policyid.$tokenname" \
--minting-script-file $script \
--metadata-json-file metadata.json  \
--invalid-hereafter $slotnumber \
--witness-override 2 \
--out-file matx.raw
```
## 7. Sign Transaksi
Untuk melakukan sign pada transaksii dibutuhkan `.skey` dari wallet pengirim
```
cardano-cli transaction sign  \
--signing-key-file /home/ivannizar/cardano-src/cardano-node/myaddress/address1.skey  \
--signing-key-file policy/policy.skey  \
--mainnet --tx-body-file matx.raw  \
--out-file matx.signed
```
## 8. Submit Transaksi
```
cardano-cli transaction submit --tx-file matx.signed $testnet
```
Output: Transaction successfully submitted.

## 9. Cek Wallet Penerima
![image](https://user-images.githubusercontent.com/71221969/215038569-28ad75d2-f3d6-449c-ae03-8b173a785e36.png)
Sudah terdapat token pada wallet penerima
