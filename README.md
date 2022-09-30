<h1 align="center">Ödüllü Ödüllü Ollo Testneti Kurulum Rehberi

## Merhabalar, bugün Ollo Station testnetine katılıyor olacağız. Sağ üstten yıldızlayıp forklamayı unutmayalım.

![image](https://user-images.githubusercontent.com/101462877/192958500-67eec3e2-ba83-48c6-ba21-182c037c11ad.png)

## Ödül detaylarını aşağıdaki görselde bulabilirsiniz. Sorularınız olursa: [LossNode Chat](https://t.me/LossNode)

![image](https://user-images.githubusercontent.com/101462877/192959733-27721499-dd71-4b9e-9071-4548fbd10351.png)

## Sistem gereksinimleri:
NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Testnet | 4          | 8         | 100  |

## Ollo Station için önemli linkler:
- [Website](https://www.ollostation.zone/)
- [Explorer](http://explorer.stavr.tech/ollo/)
- [Twitter](https://twitter.com/OLLOStation)
- [Discord](https://discord.gg/eVsKcYANPU)

# 1a) Script ile kurulum.

```
wget -O ollo.sh https://raw.githubusercontent.com/thisislexar/Ollo-Odullu-Testnet/main/ollo.sh && chmod +x ollo.sh && ./ollo.sh
```




# 1b) Manuel kurulum.

Node bilginizi geliştirmek adına dilerseniz [Manuel Kurulum](https://github.com/thisislexar/Ollo-Odullu-Testnet/blob/main/ollo_manual.md) da yapabilirsiniz.


# 2) Devam edelim. 

## Sync durumunu kontrol etmek için:

```
ollod status 2>&1 | jq .SyncInfo
``` 

## Cüzdan oluşturalım.
```
ollod keys add <CÜZDANADI>
``` 
Var olan bir cüzdanı kullanmak isterseniz:

```
ollod keys add <CÜZDANADI> --recover
``` 

## [Discord](https://discord.gg/eVsKcYANPU)'a giderek faucet alalım.

```
!request <CÜZDANADRESİ>
```

![image](https://user-images.githubusercontent.com/101462877/192987942-0b6da39f-3393-4a70-a442-07f858aaf4b9.png)


## [Explorer](http://explorer.stavr.tech/ollo/)'dan cüzdanımıza token geldiğini kontrol edelim.

![image](https://user-images.githubusercontent.com/101462877/192988540-f81fd198-0005-4c5d-be43-4e29b31431b2.png)

## Validator oluşturalım.


```
ollod tx staking create-validator \
  --amount 10000000utollo \
  --from <CÜZDANADI> \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(ollod tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id ollo-testnet-0 \
  --website="http://linktr.ee/LossNode" \
  --details="Testing the Ollo"
```


# Bazı komutlar:

Log kontrolü

```
journalctl -fu ollod -o cat
```


Servisi durdurma

```
sudo systemctl stop ollod
```

Servisi tekrar başlatma

```
sudo systemctl restart ollod
```

Token delege etme

```
ollod tx staking delegate ollovaloper1tdp45gtcfujsskg75k7tlxxxxxxx 10000000utollo --chain-id=ollo-testnet-0  --from <CÜZDANADI>
```

Validator düzenleme

```
ollod tx staking edit-validator \
  --moniker=$NODENAME \
  --identity="<KEYBASE ID'NİZ>" \
  --website="<WEBSİTE LİNKİ>" \
  --details="AÇIKLAMA" \
  --chain-id=ollo-testnet-0  \
  --from=<CÜZDANADI>
``` 


# Node silmek için:

```
sudo systemctl stop ollod
sudo systemctl disable ollod
sudo rm /etc/systemd/system/ollo* -rf
sudo rm $(which ollod) -rf
sudo rm $HOME/.ollo* -rf
sudo rm $HOME/ollo -rf
sed -i '/OLLO_/d' ~/.bash_profile
``` 
