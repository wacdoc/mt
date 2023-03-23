# Ibni s-server tiegħek li jibgħat il-posta SMTP

## preambolu

SMTP jista’ jixtri direttament servizzi minn bejjiegħa tal-cloud, bħal:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali sħaba email push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Tista 'wkoll tibni s-server tal-posta tiegħek stess - jibgħat illimitat, spiża ġenerali baxxa.

Hawn taħt, aħna nuru pass pass kif nibnu s-server tal-posta tagħna stess.

## Għażla tas-server

Is-server SMTP awto-ospitat jeħtieġ IP pubbliku bil-portijiet 25, 456, u 587 miftuħa.

Sħab pubbliċi użati b'mod komuni bblokkaw dawn il-portijiet b'mod awtomatiku, u jista 'jkun possibbli li jinfetħu billi toħroġ ordni ta' xogħol, iżda wara kollox hija idejqek ħafna.

Nirrakkomanda li tixtri minn host li għandu dawn il-portijiet miftuħa u jappoġġja t-twaqqif ta 'ismijiet ta' domain reverse.

Hawnhekk, nirrakkomanda [Contabo](https://contabo.com) .

Contabo huwa fornitur ta 'hosting ibbażat fi Munich, il-Ġermanja, imwaqqaf fl-2003 bi prezzijiet kompetittivi ħafna.

Jekk tagħżel l-Euro bħala l-munita tax-xiri, il-prezz ikun irħas (server b'memorja ta '8GB u 4 CPUs jiswa madwar 529 wan fis-sena, u l-miżata tal-installazzjoni inizjali hija b'xejn għal sena).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Meta tagħmel ordni, rimarka `prefer AMD` , u s-server b'AMD CPU se jkollu prestazzjoni aħjar.

F'dan li ġej, se nieħu l-VPS ta' Contabo bħala eżempju biex nuri kif tibni s-server tal-posta tiegħek stess.

## Konfigurazzjoni tas-sistema Ubuntu

Is-sistema operattiva hawnhekk hija Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Jekk is-server fuq ssh juri `Welcome to TinyCore 13!` (kif muri fil-figura hawn taħt), dan ifisser li s-sistema għadha ma ġietx installata. Jekk jogħġbok aqla ssh u stenna għal ftit minuti biex terġa' tidħol.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Meta tidher `Welcome to Ubuntu 22.04.1 LTS` , l-inizjalizzazzjoni tkun kompluta, u tista 'tkompli bil-passi li ġejjin.

### [Mhux obbligatorju] Inizjalizza l-ambjent tal-iżvilupp

Dan il-pass huwa fakultattiv.

Għall-konvenjenza, nressaq l-installazzjoni u l-konfigurazzjoni tas-sistema tas-softwer ubuntu [f'github.com/wactax](https://github.com/wactax/ops.os/tree/main/ubuntu) /ops.os/tree/main/ubuntu .

Mexxi l-kmand li ġej biex tinstalla bi klikk waħda.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Utenti Ċiniżi, jekk jogħġbok uża l-kmand li ġej minflok, u l-lingwa, iż-żona tal-ħin, eċċ se jiġu ssettjati awtomatikament.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo jippermetti l-IPV6

Ippermetti IPV6 sabiex l-SMTP ikun jista' jibgħat ukoll emails b'indirizzi IPV6.

editja `/etc/sysctl.conf`

Immodifika jew żid il-linji li ġejjin

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Segwi bit [-tutorja ta' contabo: Żid il-konnettività IPv6 mas-server tiegħek](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Editja `/etc/netplan/01-netcfg.yaml` , żid ftit linji kif muri fil-figura hawn taħt (il-fajl tal-konfigurazzjoni default Contabo VPS diġà għandu dawn il-linji, sempliċement neħħihom il-kumment).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Imbagħad `netplan apply` biex il-konfigurazzjoni modifikata tidħol fis-seħħ.

Wara li l-konfigurazzjoni tirnexxi, tista 'tuża `curl 6.ipw.cn` biex tara l-indirizz ipv6 tan-netwerk estern tiegħek.

## Ikklonja l-ops tar-repożitorju tal-konfigurazzjoni

```
git clone https://github.com/wactax/ops.soft.git
```

## Iġġenera ċertifikat SSL b'xejn għall-isem tad-dominju tiegħek

Li tibgħat posta teħtieġ ċertifikat SSL għall-encryption u l-iffirmar.

Aħna nużaw [acme.sh](https://github.com/acmesh-official/acme.sh) biex niġġeneraw ċertifikati.

acme.sh hija għodda awtomatizzata għall-iffirmar taċ-ċertifikati,

Daħħal il-maħżen tal-konfigurazzjoni ops.soft, mexxi `./ssl.sh` , u se jinħoloq folder `conf` fid **-direttorju ta 'fuq** .

Sib il-fornitur tad-DNS tiegħek minn [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , editja `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Imbagħad mexxi `./ssl.sh 123.com` biex tiġġenera ċertifikati `123.com` u `*.123.com` għall-isem tad-dominju tiegħek.

L-ewwel ġirja awtomatikament tinstalla [acme.sh](https://github.com/acmesh-official/acme.sh) u żżid kompitu skedat għal tiġdid awtomatiku. Tista 'tara `crontab -l` , hemm linja bħal din kif ġej.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Il-mogħdija għaċ-ċertifikat iġġenerat hija xi ħaġa bħal `/mnt/www/.acme.sh/123.com_ecc。`

It-tiġdid taċ-ċertifikat se jsejjaħ script `conf/reload/123.com.sh` , editja din l-iskrittura, tista 'żżid kmandi bħal `nginx -s reload` biex jġedded il-cache taċ-ċertifikat ta' applikazzjonijiet relatati.

## Ibni server SMTP ma chasquid

[chasquid](https://github.com/albertito/chasquid) huwa server SMTP open source miktub bil-lingwa Go.

Bħala sostitut għall-programmi antiki tas-server tal-posta bħal Postfix u Sendmail, chasquid huwa aktar sempliċi u faċli biex jintuża, u huwa wkoll aktar faċli għall-iżvilupp sekondarju.

Mexxi `./chasquid/init.sh 123.com` se jiġi installat awtomatikament bi klikk waħda (issostitwixxi 123.com bl-isem tad-dominju li jibgħat).

## Ikkonfigura l-Firma tal-Email DKIM

DKIM jintuża biex jintbagħat firem email biex jipprevjeni li l-ittri jiġu ttrattati bħala spam.

Wara li l-kmand jaħdem b'suċċess, tkun imħeġġeġ biex tissettja r-rekord DKIM (kif muri hawn taħt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Żid biss rekord TXT mad-DNS tiegħek (kif muri hawn taħt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Ara l-istatus tas-servizz u zkuk

 `systemctl status chasquid` Ara l-istatus tas-servizz.

L-istat ta 'tħaddim normali huwa kif muri fil-figura hawn taħt

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` jew `journalctl -xeu chasquid` jistgħu jaraw ir-reġistru tal-iżbalji.

## Reverse konfigurazzjoni tal-isem tad-dominju

L-isem tad-dominju invers huwa li jippermetti li l-indirizz IP jiġi solvut għall-isem tad-dominju korrispondenti.

L-issettjar ta' isem ta' dominju invers jista' jipprevjeni emails milli jiġu identifikati bħala spam.

Meta tiġi riċevuta l-posta, is-server li jirċievi jwettaq analiżi tal-isem tad-dominju invers fuq l-indirizz IP tas-server li jibgħat biex jikkonferma jekk is-server li jibgħat għandux isem tad-dominju reverse validu.

Jekk is-server li jibgħat m'għandux isem tad-dominju invers jew jekk l-isem tad-dominju invers ma jaqbilx mal-indirizz IP tas-server li jibgħat, is-server li jirċievi jista' jagħraf l-email bħala spam jew jirrifjutah.

Żur [https://my.contabo.com/rdns](https://my.contabo.com/rdns) u kkonfigura kif muri hawn taħt

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Wara li ssettja l-isem tad-dominju b'lura, ftakar li tikkonfigura r-riżoluzzjoni 'l quddiem tal-isem tad-dominju ipv4 u ipv6 lis-server.

## Editja l-isem tal-host ta' chasquid.conf

Immodifika `conf/chasquid/chasquid.conf` għall-valur tal-isem tad-dominju invers.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Imbagħad ħaddem `systemctl restart chasquid` biex terġa 'tibda s-servizz.

## Backup conf għar-repożitorju git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Pereżempju, nagħmel backup tal-folder tal-conf fil-proċess tal-github tiegħi stess kif ġej

Oħloq maħżen privat l-ewwel

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Daħħal id-direttorju tal-konf u ibgħat lill-maħżen

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Żid il-mittent

run

```
chasquid-util user-add i@wac.tax
```

Jista 'jżid il-mittent

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Ivverifika li l-password hija ssettjata b'mod korrett

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Wara li żżid l-utent, `chasquid/domains/wac.tax/users` se jiġu aġġornati, ftakar li tissottomettiha lill-maħżen.

## DNS żid rekord SPF

SPF (Sender Policy Framework) hija teknoloġija ta' verifika tal-email użata biex tipprevjeni l-frodi tal-email.

Jivverifika l-identità ta’ mittent tal-posta billi jiċċekkja li l-indirizz IP ta’ min jibgħat jaqbel mar-rekords DNS tal-isem tad-dominju li jiddikjara li hu, u jipprevjeni lill-frodisti milli jibagħtu emails foloz.

Iż-żieda ta 'rekords SPF tista' tipprevjeni emails milli jiġu identifikati bħala spam kemm jista 'jkun.

Jekk is-server tal-isem tad-dominju tiegħek ma jappoġġjax it-tip SPF, żid biss ir-rekord tat-tip TXT.

Pereżempju, l-SPF ta ' `wac.tax` huwa kif ġej

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF għal `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Innota li għandi `include:_spf.google.com` hawn, dan għaliex ser nikkonfigura `i@wac.tax` bħala l-indirizz li jibgħat fil-kaxxa postali ta' Google aktar tard.

## Konfigurazzjoni DNS DMARC

DMARC hija l-abbrevjazzjoni ta' (Awtentikazzjoni, Rappurtar u Konformità tal-Messaġġ ibbażati fuq id-Dominju).

Jintuża biex jinqabad SPF bounces (forsi kkawżati minn żbalji ta 'konfigurazzjoni, jew xi ħadd ieħor qed jippretendi li inti biex tibgħat spam).

Żid rekord TXT `_dmarc` ,

Il-kontenut huwa kif ġej

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

It-tifsira ta 'kull parametru hija kif ġej

### p (Politika)

Jindika kif timmaniġġja l-emails li jfallu mill-verifika SPF (Sender Policy Framework) jew DKIM (DomainKeys Identified Mail). Il-parametru p jista' jiġi ssettjat għal wieħed minn tliet valuri:

* xejn: Ma tittieħed l-ebda azzjoni, ir-riżultat tal-verifika biss jingħata lura lill-mittent permezz tal-mekkaniżmu ta' rappurtar bl-email.
* Kwarantina: Poġġi l-posta li ma għaddietx mill-verifika fil-folder tal-ispam, iżda mhux se tirrifjuta l-posta direttament.
* tirrifjuta: Tiċħad direttament emails li ma rnexxilhomx verifika.

### fo (Għażliet ta' Falliment)

Jispeċifika l-ammont ta' informazzjoni mibgħuta lura mill-mekkaniżmu ta' rappurtar. Jista' jiġi ssettjat għal wieħed mill-valuri li ġejjin:

* 0: Irrapporta r-riżultati tal-validazzjoni għall-messaġġi kollha
* 1: Irrapporta biss messaġġi li jfallu l-verifika
* d: Irrapporta biss il-fallimenti tal-verifika tal-isem tad-dominju
* s: jirrapporta biss in-nuqqasijiet tal-verifika SPF
* l: Irrapporta biss il-fallimenti tal-verifika DKIM

### rua & ruf

* rua (Rapporting URI for Aggregate reports): Indirizz elettroniku biex tirċievi rapporti aggregati
* ruf (Rapporting URI għal rapporti Forensiċi): indirizz elettroniku biex tirċievi rapporti dettaljati

## Żid rekords MX biex tibgħat emails lil Google Mail

Minħabba li ma stajtx insib kaxxa postali korporattiva b'xejn li tappoġġja indirizzi universali (Catch-All, tista' tirċievi kwalunkwe email mibgħuta lil dan l-isem tad-dominju, mingħajr restrizzjonijiet fuq il-prefissi), użajt chasquid biex tibgħat l-emails kollha lill-kaxxa tal-posta tal-Gmail tiegħi.

**Jekk għandek il-kaxxa postali tan-negozju mħallsa tiegħek stess, jekk jogħġbok timmodifikax l-MX u aqbeż dan il-pass.**

Editja `conf/chasquid/domains/wac.tax/aliases` , issettja l-kaxxa tal-posta li tibgħat

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` jindika l-emails kollha, `i` huwa l-prefiss tal-indirizz tal-email tal-utent li jibgħat maħluq hawn fuq. Biex tibgħat il-posta, kull utent jeħtieġ li jżid linja.

Imbagħad żid ir-rekord MX (nippunta direttament lejn l-indirizz tal-isem tad-dominju invers hawn, kif muri fl-ewwel linja fil-figura hawn taħt).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Wara li titlesta l-konfigurazzjoni, tista' tuża indirizzi email oħra biex tibgħat emails lil `i@wac.tax` u `any123@wac.tax` biex tara jekk tistax tirċievi emails fil-Gmail.

Jekk le, iċċekkja l-log chasquid ( `grep chasquid /var/log/syslog` ).

## Ibgħat email lil i@wac.tax bil-Google Mail

Wara li Google Mail irċieva l-posta, naturalment ttamat li nwieġeb `i@wac.tax` minflok b'i.wac.tax@gmail.com.

Żur [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) u kklikkja "Żid indirizz elettroniku ieħor".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Imbagħad, daħħal il-kodiċi ta 'verifika li waslet bl-email li ġiet mibgħuta lil.

Fl-aħħarnett, jista 'jiġi ssettjat bħala l-indirizz default tal-mittent (flimkien mal-għażla li tirrispondi bl-istess indirizz).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

B'dan il-mod, lestejna l-istabbiliment tas-server tal-posta SMTP u fl-istess ħin nużaw Google Mail biex nibagħtu u nirċievu emails.

## Ibgħat email tat-test biex tivverifika jekk il-konfigurazzjoni hix ta' suċċess

Daħħal `ops/chasquid`

Mexxi `direnv allow` li tinstalla dipendenzi (direnv ġie installat fil-proċess ta 'inizjalizzazzjoni ta' ċavetta waħda preċedenti u ġie miżjud ganċ mal-qoxra)

imbagħad run

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

It-tifsira tal-parametri hija kif ġej

* utent: isem tal-utent SMTP
* pass: password SMTP
* lil: destinatarju

Tista' tibgħat email tat-test.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Huwa rakkomandat li tuża Gmail biex tirċievi emails tat-test biex tivverifika jekk il-konfigurazzjonijiet humiex ta' suċċess.

### Encryption standard TLS

Kif muri fil-figura hawn taħt, hemm dan il-lock żgħir, li jfisser li ċ-ċertifikat SSL ġie attivat b'suċċess.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Imbagħad ikklikkja "Uri l-Email Oriġinali"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Kif muri fil-figura hawn taħt, il-paġna tal-posta oriġinali tal-Gmail turi DKIM, li jfisser li l-konfigurazzjoni DKIM hija ta 'suċċess.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Iċċekkja l-Ricevut fl-intestatura tal-email oriġinali, u tista 'tara li l-indirizz tal-mittent huwa IPV6, li jfisser li IPV6 huwa kkonfigurat ukoll b'suċċess.
