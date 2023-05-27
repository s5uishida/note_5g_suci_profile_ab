# A Note for 5G SUCI Profile A/B Scheme

<h2 id="open5gs">udm.yaml of Open5GS</h2>

As of 2023.05.14, for the setting of Home Network Public Key for 5G SUCI Profile A/B Scheme, the comment in [udm.yaml.in](https://github.com/open5gs/open5gs/blob/main/configs/open5gs/udm.yaml.in) of Open5GS is very helpful.
```yaml
...
#
#  <Home Network Public Key>
#
#  o Generate the private key as below.
#    $ openssl genpkey -algorithm X25519 -out /etc/open5gs/hnet/curve25519-1.key
#    $ openssl ecparam -name prime256v1 -genkey -conv_form compressed -out /etc/open5gs/hnet/secp256r1-2.key
#
#  o The private and public keys can be viewed with the command.
#    The public key is used when creating the SIM.
#    $ openssl pkey -in /etc/open5gs/hnet/curve25519-1.key -text
#    $ openssl ec -in /etc/open5gs/hnet/secp256r1-2.key -conv_form compressed -text
#
#  o Home network public key identifier(PKI) value : 1
#    Protection scheme identifier : ECIES scheme profile A
#  udm:
#    hnet:
#      - id: 1
#        scheme: 1
#        key: /etc/open5gs/hnet/curve25519-1.key
#
#  o Home network public key identifier(PKI) value : 2
#    Protection scheme identifier : ECIES scheme profile B
#  udm:
#    hnet:
#      - id: 2
#        scheme: 2
#        key: /etc/open5gs/hnet/secp256r1-2.key
#
...
```
For example, from `curve25519-1.key`, get the public key to be set to `homeNetworkPublicKey` in the UE configuration of UERANSIM as follows.
```
# openssl pkey -in curve25519-1.key -text_pub -noout | sed '/^[X25519|pub]/d' | tr -d "\n: " | sed '$a\'
e421686f6fb2d70e3fa28d940494095686c3179fef53514667a6ed106b8a7d3d
```

<h2 id="free5GC">udmcfg.yaml of free5GC</h2>

For free5GC, there are setting items in [udmcfg.yaml](https://github.com/free5gc/free5gc/blob/main/config/udmcfg.yaml).
```yaml
...
  # test data set from TS33501-f60 Annex C.4
  SuciProfile: # Home Network Public Key ID = slice index +1
    - ProtectionScheme: 1 # Protect Scheme: Profile A
      PrivateKey: c53c22208b61860b06c62e5406a7b330c2b577aa5558981510d128247d38bd1d
      PublicKey: 5a8d38864820197c3394b92613b20b91633cbd897119273bf8e4a6f4eec0a650
    - ProtectionScheme: 2 # Protect Scheme: Profile B
      PrivateKey: F1AB1074477EBCC7F554EA1C5FC368B1616730155E0041AC447D6301975FECDA
      PublicKey: 0472DA71976234CE833A6907425867B82E074D44EF907DFB4B3E21C1C2256EBCD15A7DED52FCBB097A4ED250E036C7B9C8C7004C4EEDC4F068CD7BF8D3F900E3B4
...
```

<h2 id="ueransim">xxx-ue.yaml of UERANSIM</h2>

In addition, [UERANSIM](https://github.com/aligungr/UERANSIM/tree/master/config) supported 5G SUCI Profile A Scheme on 2023.05.09.
UERANSIM can use 5G SUCI Profile A Scheme with Open5GS and free5GC.
```yaml
...
# SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
protectionScheme: 0
# Home Network Public Key for protecting with SUCI Profile A
homeNetworkPublicKey: '5a8d38864820197c3394b92613b20b91633cbd897119273bf8e4a6f4eec0a650'
# Home Network Public Key ID for protecting with SUCI Profile A
homeNetworkPublicKeyId: 1
# Routing Indicator
routingIndicator: '0000'
...
```
