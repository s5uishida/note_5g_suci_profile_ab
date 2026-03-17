# A Note for 5G SUCI Profile A/B Scheme

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="open5gs"></a>

## udm.yaml of Open5GS

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

<a id="open5gs_a"></a>

### Profile A

For example, from `curve25519-1.key`, get the public key to be set to `homeNetworkPublicKey` in the UE configuration of UERANSIM and PacketRusher as follows.
```
# openssl pkey -in curve25519-1.key -text_pub -noout 2> /dev/null | sed -n '/^pub:/,+3p' | tail -3 | tr -d "\n: " | sed '$a\'
e421686f6fb2d70e3fa28d940494095686c3179fef53514667a6ed106b8a7d3d
```

<a id="open5gs_b"></a>

### Profile B

<a id="open5gs_b_compress_pub_key"></a>

#### Get compressed public key

For example, from `secp256r1-2.key`, get the compressed public key as follows.
```
# openssl ec -in secp256r1-2.key -conv_form compressed -text -noout 2> /dev/null | sed -n '/^pub:/,+3p' | tail -3 | tr -d "\n: " | sed '$a\'
03adefcd1317d1ce8562ec25b91b4800120e1236d6e2661ea4235a84e3c85da244
```

<a id="open5gs_b_uncompress_pub_key"></a>

#### Get uncompressed public key

For example, from `secp256r1-2.key`, get the uncompressed public key to be set to `homeNetworkPublicKey` in the UE configuration of PacketRusher as follows.
```
# openssl ec -in secp256r1-2.key -conv_form uncompressed -text -noout 2> /dev/null | sed -n '/^pub:/,+5p' | tail -5 | tr -d "\n: " | sed '$a\'
04adefcd1317d1ce8562ec25b91b4800120e1236d6e2661ea4235a84e3c85da244bc42a594cf5612f49e8fbe2857d8e499f91322c737fccf1bbdb6e4d424a80a95
```

<a id="free5GC"></a>

## udmcfg.yaml of free5GC

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

<a id="ueransim"></a>

## xxx-ue.yaml of UERANSIM

[UERANSIM](https://github.com/aligungr/UERANSIM/tree/master/config) supported 5G SUCI Profile A Scheme on 2023.05.09.
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

<a id="packetrusher"></a>

## config.yml of PacketRusher

[PacketRusher](https://github.com/HewlettPackard/PacketRusher/tree/main/config) supports 5G SUCI Profile A/B Schemes (uncompressed public keys).
```yaml
...
# In 5G, the UE's identity to the AMF as a SUCI (Subscription Concealed Identifier)
#
# SUCI format is suci-<supi_type>-<MCC>-<MNC>-<routing_indicator>-<protection_scheme>-<public_key_id>-<scheme_output>
# With default configuration, SUCI sent to AMF will be suci-0-999-70-0000-0-0-0000000120
#
# SUCI Routing Indicator allows the AMF to route the UE to the correct UDM
routingindicator: "0000"
#
# SUCI Protection Scheme: 0 for Null-scheme, 1 for Profile A and 2 for Profile B
protectionScheme: 0
#
# Home Network Public Key
# Ignored with default Null-Scheme configuration
homeNetworkPublicKey: "5a8d38864820197c3394b92613b20b91633cbd897119273bf8e4a6f4eec0a650"
#
# Home Network Public Key ID
# Ignored ith default Null-Scheme configuration
homeNetworkPublicKeyID: 1
...
```
