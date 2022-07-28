# DANCE - DANE Authentication for Network Clients Everywhere
# Problem Statement: We’re missing DNS/DNSSEC support for finding, identifying, and authenticating “Digital Identity Trust Registries”. Here is our proposal for solving this. 

READ the PDF version of this with more details: https://github.com/CIRALabs/DANCE/blob/main/trust-registry.pdf


# Leveraging DNSSEC in Digital Identity (short)

This is an extract of the DNS queries performed for this concept:

* $ dig 5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca tlsa +dnssec +multi
* $	dig iotregistry.ca tlsa +dnssec +multi
* $	dig iotregistry.ca TXT +dnssec +multi  (for now until we have a TR record :) )
* $	dig iotregistry.ca._trustregistry.trustregistry.ca tlsa +dnssec +multi

By leveraging TLSA records, PKI, and the existing DNS/DNSSEC infrastructure, it is possible to validate the authenticity and integrity of an end entity as well as its issuer/authority, simply by performing a handful of DNS queries and manipulating the results therein.   

To perform a DNS lookup and retrieve the corresponding TLSA records to verify an end entity (a device in this case), we perform a dig query with: 


## $ dig 5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca tlsa +dnssec +multi

; <<>> DiG 9.16.1-Ubuntu <<>> 5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca tlsa +dnssec +multi
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12308
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4000
;; QUESTION SECTION:
;5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca. IN        TLSA

This query returns 3 important answers:

1.	The first answer in the dig query in Figure 1.2 contains the TLSA record usage 3 0 1 corresponding to our IoT device. This TLSA usage indicates the record corresponds to an end entity, and we are including an unsalted SHA256 hash of its binary encoded certificate data. This can then be used by another entity to validate that the certificate presented by a device claiming to be 5672476c-e9aa-4aa3-9cfd-2aef76f44f0f is authoritatively valid by checking that the SHA256 hash of the presented certificate matches its TLSA record.
2.	For the sake of demonstration, we have included a TLSA record of type 3 0 0 for the device. See the second answer in Figure 1.2. In an operational setting, posting the device certificate in full would not be necessary. However, in this case by posting the device certificate we can allow readers to follow along with the demo themselves. Included later in this document are the steps necessary to convert this hex encoded certificate data back into a PEM formatted cert, but for brevity we have presented the decoded cert below (see Figure 2)
3.	The third and final answer in the query is the RRSIG corresponding to the 5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca record set (see Figure 1.2), indicating that zone is secured with DNSSEC. 
 
 ;; ANSWER SECTION:
5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca. 2112 IN TLSA 3 0 0 (
                                308204B8308203A0A00302010202021002300D06092A
                                864886F70D01010B0500305B310B3009060355040613
                                0243413110300E06035504080C074F6E746172696F31
                                0D300B060355040A0C0443495241310D300B06035504
                                0B0C044C616273311C301A06035504030C13496F5420
                                526567697374727920537562204341301E170D323230
                                3732313137333535365A170D32333037333131373335
                                35365A307B310B30090603550406130243413110300E
                                06035504080C074F6E746172696F310D300B06035504
                                0A0C0443495241310D300B060355040B0C044C616273
                                313C303A06035504030C3335363732343736632D6539
                                61612D346161332D396366642D326165663736663434
                                6630662E696F7472656769737472792E636130820122
                                300D06092A864886F70D01010105000382010F003082
                                010A0282010100D0CF04E664F5544A04D815EA393336
                                1A4036CF09E2E543B9717CB064F3D7A599E8DEB2E6E5
                                680831DFE58F4CDB5F2592DA70C2AFE82987095EA54B
                                F865F6A8960F2720BCC67732858C32725B7968D463D6
                                54CCAC4D12CF46F755B167EF1F47F35BC91C90E58660
                                0B1034551F395163E1241E549BC49DD5A1EC38562C42
                                81686EB6DB79ED7EBD14353C8D429D90ED844AB2F45F
                                38EA51699E2916C43AB8FB712B3EA59F3A4D1EB1F5EE
                                F5BB0765A347FA0BA95EB9F33F736FB8B32EEF22D2C7
                                9C3D54C001E0FC78F61544D1A11A0FD4DF06B595DCA8
                                E38A726CE825A9C13C872707BB7B71E9865F3B71A711
                                C94792C765EDFDD8E077B02949FC1DCB8DD68A1A9302
                                03010001A38201643082016030090603551D13040230
                                00301106096086480186F84201010404030205A03033
                                06096086480186F842010D042616244F70656E53534C
                                2047656E65726174656420436C69656E742043657274
                                69666963617465301D0603551D0E04160414EA72BC85
                                779CFD67D343F8AE097DC142607070DC301F0603551D
                                23041830168014F8570ED695CE40706FD794970E6414
                                0E85B55FE0300E0603551D0F0101FF0404030205E030
                                1D0603551D250416301406082B060105050703020608
                                2B0601050507030430819B0603551D11048193308190
                                823B35363732343736632D653961612D346161332D39
                                6366642D3261656637366634346630662E5F64657669
                                63652E696F7472656769737472792E63618239353637
                                32343736632D653961612D346161332D396366642D32
                                61656637366634346630662E5F6465766963652E7061
                                726B6F736572762E636181166A6163717565732E6C61
                                746F757240636972612E6361300D06092A864886F70D
                                01010B050003820101001E47C2C6E1D4718E408C0B7D
                                A149A4BECDDB36178B738A8532DED4C54A9421A0FDE7
                                BB22BD0F804E476D6817CD91E0EFA575AA11A85A5D08
                                6DF553B0C4F7092C0D281C534FBEAE7094E3E2262600
                                A91376A23EA2C7DF2889D9BB74FD48C9221FA7E348E4
                                F3889F8B28102F09DEB6FBF5BF36637AC2FDDCC3A16A
                                CBA180D878323DE79BED7CCDE7D762A3CC3DF94C2B30
                                AAE5585D9A03C76C1F2067767063F7EDE5A7CEC7FEAC
                                9DCCBE5141C097FA22C2B6C3390FA3E184D856893556
                                D26A2CDB50F39B2433415633433A30DB96CE88231EF7
                                5ADA6F0274217AA36263F2DF7821296BBDD783372CD2
                                CDA7CAE14C9C43700E07EE935249524312AEF266B9FF
                                773E )
5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca. 2112 IN TLSA 3 0 1 (
                                2ADF8C61777F73838D04FAF402992689419B67421708
                                1C9A930D64E1FF5677CE )
5672476c-e9aa-4aa3-9cfd-2aef76f44f0f._device.iotregistry.ca. 2112 IN RRSIG TLSA 13 4 3600 (
                                20220811000000 20220721000000 11926 iotregistry.ca.
                                mZ2S9G0O/+6WzWj0PgSx26BWrQQIeiCcsxlc3UV7YfQK
                                vihQtnW7+mhByBHncHBZAb0c6Vh+lP0v1QIno6EPAQ== )

;; Query time: 17 msec
;; SERVER: 10.4.91.26#53(10.4.91.26)
;; WHEN: Thu Jul 28 09:18:07 EDT 2022
;; MSG SIZE  rcvd: 1472

To perform a DNS lookup to retrieve the corresponding TLSA records and certificates to validate the cryptographic chain of trust, we perform a dig query on iotregistry.ca: 

## $ dig iotregistry.ca tlsa +dnssec +multi

; <<>> DiG 9.16.1-Ubuntu <<>> iotregistry.ca tlsa +dnssec +multi
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28126
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4000
;; QUESTION SECTION:
;iotregistry.ca.                IN TLSA
 
This query returns 3 important answers:

1.	The first answer to our query is the IoT Registry sub certificate (see Figure 3.2). The certificate content is hexadecimal (DER) encoded as required by https://datatracker.ietf.org/doc/html/rfc6698.
2.	The second answer to our query is the IoT Registry root certificate (see Figure 3.3).
3.	The third and final answer in the query is the RRSIG corresponding to the iotregistry.ca record set, indicating that zone is secured with DNSSEC (see Figure 3.3)

;; ANSWER SECTION:
iotregistry.ca.         3600 IN TLSA 0 0 0 (
                                308203C4308202ACA00302010202144BBB302E1E9F78
                                312998E9DA59373C3C358499A0300D06092A864886F7
                                0D01010B0500305C310B300906035504061302434131
                                10300E06035504080C074F6E746172696F310D300B06
                                0355040A0C0443495241310D300B060355040B0C044C
                                616273311D301B06035504030C14496F542052656769
                                7374727920526F6F74204341301E170D323230373135
                                3138313331335A170D3432303731303138313331335A
                                305C310B30090603550406130243413110300E060355
                                04080C074F6E746172696F310D300B060355040A0C04
                                43495241310D300B060355040B0C044C616273311D30
                                1B06035504030C14496F542052656769737472792052
                                6F6F7420434130820122300D06092A864886F70D0101
                                0105000382010F003082010A0282010100C77F6F2A13
                                3F1E9FF7C5136CD9D2743FE84AC3C3AFA63923425643
                                74C21670FA40528AE4C7C1A047DDCB636891907D328B
                                FDA29C2C435DAC1B006817544FB2EAA27AD6639FB15F
                                3AAF5BEAC5D58BB1F50200AB89F2315743E9DC18A447
                                881A10630FBF7FD207AA4DCF44E2BE872412BBAA81C4
                                EF5B0DCFE020611E50ED5AC0AFFB574FA7C0C64D39C9
                                14021B5C940B9D4B95F8DBD84AAB998FAF053F8B26E2
                                428EA92F6C7371AC13F4220AFEF13296BF137FF95D85
                                C410A5CBB658770A2B8049E9E9EE12B276E4EB0227DC
                                972AC891D69D7F4776F5E37F7BC2A2751F90174E850A
                                9BFBC928808E9C8990DEB5D1714B92148E053731EFC7
                                AB8E95DAC0423E2D050203010001A37E307C301D0603
                                551D0E0416041418E08E5526B9BB490250AC7A9407FE
                                35D7A9DB56301F0603551D2304183016801418E08E55
                                26B9BB490250AC7A9407FE35D7A9DB56300F0603551D
                                130101FF040530030101FF300E0603551D0F0101FF04
                                040302018630190603551D1104123010820E696F7472
                                656769737472792E6361300D06092A864886F70D0101
                                0B0500038201010017BFF33253A2DFF1CD5A118A8A2E
                                7E348B1DD7EFD87DDD58508026B1E9A0638A61168B73
                                793912618DB7E09181EAB91E58AF997D9BB8CF74BBAF
                                62965F824497BFD35BCA0CD486148B0550F9C523B188
                                973AF8501A2EB5BAF918D4380320166107E73580BDCC
                                2DDC3EEDA58932EE3388954CED3AD68BF612F741325A
                                EB2BF13D44CE4479B4D48F06733810FC0541A707677E
                                46ACBD11A04C8043A3ED3078D551E2911CC68EE2CA03
                                947BEFD8CE868938D0A19D1AA1C00F5BA967E21E2400
                                4997B6D0D820AF6F7AF89E7EF653BAC981D241769063
                                53F7A4223075B6EC3A9D90E4EE772DF387DAF3779DE8
                                D488E8479B9DED2A7B55B2D0720BE41DA05CC409457D )
iotregistry.ca.         3600 IN TLSA 0 0 0 (
                                308203B53082029DA00302010202021000300D06092A
                                864886F70D01010B0500305C310B3009060355040613
                                0243413110300E06035504080C074F6E746172696F31
                                0D300B060355040A0C0443495241310D300B06035504
                                0B0C044C616273311D301B06035504030C14496F5420
                                526567697374727920526F6F74204341301E170D3232
                                303731353138333534345A170D333230373132313833
                                3534345A305B310B3009060355040613024341311030
                                0E06035504080C074F6E746172696F310D300B060355
                                040A0C0443495241310D300B060355040B0C044C6162
                                73311C301A06035504030C13496F5420526567697374
                                72792053756220434130820122300D06092A864886F7
                                0D01010105000382010F003082010A0282010100CCF9
                                C2730BE02B7A6A68D32D15078D5DBF8BA8351F529C82
                                5AD407FA814C58F52300AD2A0D3A6A9C6D52381B9FCB
                                3C4BD9CB5B5FAFC9935298D77E0DA74F62768DEA3092
                                35D6402374EC921241727ABAFF6FD8AB0EEA26F97C11
                                AF680E5F879A78664CA9A267BD36F4E01374D40498D2
                                BDAAF1CBDA9A5B7233AC960103C3F372FD92942A441C
                                5029EA4E17429CF6DBA9E6117B7561795DDD07960CC9
                                0A8DAD5F8DA64F6FC8E8071E28F2039AC04E2B746D5C
                                64A1515A295B30E3C396E5327F9B6FDC36768C683EB9
                                27E8497E77F822CCF33117791459EAA0ABD7240DD464
                                CC8968CA27537E400C7A43EA1A1B41F3BE7A0B17EA3A
                                524026BE75CBE70E2E0756F10203010001A38181307F
                                301D0603551D0E04160414F8570ED695CE40706FD794
                                970E64140E85B55FE0301F0603551D23041830168014
                                18E08E5526B9BB490250AC7A9407FE35D7A9DB563012
                                0603551D130101FF040830060101FF020100300E0603
                                551D0F0101FF04040302018630190603551D11041230
                                10820E696F7472656769737472792E6361300D06092A
                                864886F70D01010B0500038201010070D637533AE741
                                72A15066A6520DE01F13882E91C94875300F56F36ECB
                                BD9F210C21D91A3120D6F685B32FD2E69F503017C968
                                6F86925D15D753161364DDCF7A3E15F06F0E86EE8867
                                21A203B01853B91619F348175B83862BA4DDDEF0EC60
                                5998B8DADA63DC0C032E6E248E34EF36D118F84C525D
                                853F70E47408529724B927EAFC86205A58B7563D2AFA
                                4CCA266D17F926B864D54D96841B481F330DDAA4DEAE
                                C2F67DF4297C2447BE63BFFE196C48C85AE6AF53B280
                                ED0659F2303B53D6A657F0ECEC1C06B4F7BCD3644865
                                92D9B17310527B4C46AE158EB20E8688BFDCC879D225
                                8B31C6B4CFDFDE065C986C45348A26AA3225F1E91C2A
                                A88D130D47E8CE )
iotregistry.ca.         3600 IN RRSIG TLSA 13 2 3600 (
                                20220811000000 20220721000000 11926 iotregistry.ca.
                                E2DnG14ye0sLSk2rnq1bqYT6OL4HnqO6r/69fvL01dCR
                                xR0rVRHcSMPvaiPmuZeVsoIqMvUSVkVU0fs3KnUPPQ== )

;; Query time: 21 msec
;; SERVER: 10.4.91.26#53(10.4.91.26)
;; WHEN: Thu Jul 28 09:22:23 EDT 2022
;; MSG SIZE  rcvd: 2104



## $	dig iotregistry.ca._trustregistry.trustregistry.ca tlsa +dnssec +multi
 

; <<>> DiG 9.16.1-Ubuntu <<>> iotregistry.ca._trustregistry.trustregistry.ca tlsa +dnssec +multi
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15551
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4000
;; QUESTION SECTION:
;iotregistry.ca._trustregistry.trustregistry.ca.        IN TLSA

;; ANSWER SECTION:
iotregistry.ca._trustregistry.trustregistry.ca. 3600 IN TLSA 0 0 1 (
                                72A286E135981BB1BD7B1361B3387E4158FBABF4BBF5
                                DCBE3D6D78BB941B2CE5 )
iotregistry.ca._trustregistry.trustregistry.ca. 3600 IN RRSIG TLSA 13 5 3600 (
                                20220811000000 20220721000000 16050 trustregistry.ca.
                                L8HQcPfXvBA+mt4ptyLMb6C516+ahJemSrxbsIGUOlQf
                                HrsACqVyu3b5v+hn1qnhfo1w95ITVL5oJYf+Oy4jqA== )

;; Query time: 19 msec
;; SERVER: 10.4.91.26#53(10.4.91.26)
;; WHEN: Thu Jul 28 09:51:07 EDT 2022
;; MSG SIZE  rcvd: 234


