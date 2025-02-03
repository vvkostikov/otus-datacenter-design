# Проектная работа "Построение растянутой L2 сети между датацентрами предприятия с использованием технологии VxLAN EVPN Multi-Site."
## Презентация:
[Презентация](/labs/project/presentation/)
## Описание проекта:
Работа призвана отразить полученные навыки в проектировании сетей Датацентров с использованием тезнологии VXLAN EVPN и применением варианта построения взаимодействия между независимыми датацентрами на основе тезнологии VXLAN.

В данной задачи рассматривается проектирование адресного пространства и типовых конфигурация с использованием "шаблонов" для нескольких датацентров.

Все принципы и конфигурации описываются далее по тексту.

Работа строилась на концепции расширения вычислительного поля компании за счет строительства и подключения второго Датацентра предприятия.

С Учетом предъявляемых требования от Виртуализации и непосредственно администраторов сервисов предполагалось заложить возможность использовать как L2 так и L3 варианты взаимодействия систем между собой.

Все оборудование должно быть задублировано.

Подключение конечный устройств к коммутаторам фабрики предполагает использование технологии Multihoming EVPN.

Взаимодействие между Подразделениями на данном этапе не предполагается. В дальнейшем запланирована сборка внешнего Фаервола для организации VRF Lite.

## Ограничения стенда:

Выключен BFD и отсутсвует в конфигах с устройств. Слишком высокая нагрузка.

Финальный этап тестирования так же выполнялся с частью оборудования в выкюченном состоянии из-за нагрузки на стенд.

Так как не хватало ресурсов технология Multihoming EVPN описана в виде шаблонов конфигов, но отсутсвует в финальном выводе команд.

## Описание принципа выделения адресного пространства Underlay сети:
Порядковый номер Датацентра - DC N (Нумерация начинается с 1. "0" зарезервирован)

Loopback /32

P2P Subnets /31

loopbacks1 - Spine N-X 192.168.N0.X

loopbacks2 - Spine N-X 192.168.N1.X (зарезервирован для multicast или иных технических нужд)

loopbacks1 - Leaf N-X 192.168.N2.X

loopbacks2 - Leaf N-X 192.168.N3.X (для VTEP)

P2P 192.168.N[4-7].X/31 (Меньшее число всегда идет на Spine)

reserved 192.168.N[8-9] (зарезервированы для дальнейшего использования)

## Описание выбора номера AS для Датацентра:
Номер AS: 65NNNN,

где

NNNN - порядковый номер Датацентра.

## Описание VLAN, VNI, RD и RT:
VNI генерится на основе номера Влана в Датацентре: N0<VLAN ID в 4 значном формате>, где N это порядковый номер подразделения.

RD VLAN генерится путем составления номера AS и VNI - AS:VNI

RT VLAN на импорт и экспорт по умолчанию генерится из номера AS и VNI - AS:VNI

VRF нумеруются в соответсвии с подразделениями по порядку начиная с 1.

Им присваивается VNI для L3 маршрутиазции по формуле 1<Номер подразделения дополненный до трехзначного значения>.

Например для подразделения 1 - L3 VNI 1001

RD VRF - 65000:1001

RT VRF - 65000:1001

## Описание адресного пространства Overlay сети:
Создан VRF TENANT1 и TENANT2.

Создан 1 Влан для серверов 1 Сервиса Tenant1 - Vlan 10 - Service1 - VNI 100010 10.0.0.0/24

Создан 2 Влан для серверов 2 Сервиса Tenant1 - Vlan 20 - Service2 - VNI 100020 10.0.2.0/24

Создан 3 Влан для серверов 1 Сервиса Tenant2 - Vlan 30 - Service1 - VNI 200030 10.2.0.0/24

Создан 4 Влан для серверов 1 Сервиса Tenant2 - Vlan 40 - Service2 - VNI 200040 10.4.0.0/24

Так как мы используем для Датацентра iBGP с номером AS 65000 получаем:

Для Датацентра 1 Tenant1:

RD - 65001:100010

RT - 65001:100010

Для Датацентра 2 Tenant1:

RD - 65002:100010

RT - 65002:100010

## Описание выбора сегмента ESI и RT с LACP SYSTEM-ID:

ethernet-segmet identifier 0000:<номер датацентра в 4 значном формате>:<номер Tenant в 4 значном формате>:<номер сервиса в 4 значном формате>:<номер ESI порядковый в 4 значном формате>

route-target использует последние 12 цифр ESI в формате XX:XX:XX:XX:XX:XX

lacp system-id также использует последние 12 цифр ESI в формате XXXX.XXXX.XXXX

Соответсвенно в нашем случае для Датацентра 1 Tenant 1 Сервиса 2 ESI 1:

ethernet-segmet identifier 0000:0001:0001:0002:0001

route-target import 00:01:00:02:00:01

lacp system-id 0001.0002.0001

## Описание выбора виртуального MAC:

Чтобы избежать влияние смены MAC адреса шлюза при перемещении Любого устройства в сети, для всех LEAF с Anycast-gateway прописано использовать виртуальный MAC адрес.

ip virtual-router mac-address 0001.00d5.5dc0

## Описание DCI:
### Описание выбора RT для DCI:

Для L2 VNI, котоыре требуются растянуть между Датацентрами указывает отдельный RT  удаленного домена:

65500:VNI

Для L3 VNI не зависимо от потребностей в передачи между датацентрами используется общий RT:

65000:VNI

### Описание выбора BGW и схемы подключения:

Для целей связи между Датацентрами выбраны LEAF-1-3 и LEAF-1-4 в Датацентре 1 и LEAF-2-1 с LEAF-2-2 в Датацентре 2. Фактически они я являются Border Gateway Leaf. (BGW)

Связь обеспечивается прямыми линками между указанными коммутаторами попарно:
LEAF-1-3 - LEAF-2-2
LEAF-1-4 - LEAF-2-1

### Описание выбора адресации для связи между Датацентрами:

Решено использовать уникальную сеть 172.30.254.0/24 поделенную на /31 подсети.

Оставлен резерв для дальнейшего увеличения количества линков при возрастании тербований к отказоустойчивости или пропускной способности.

## Схема сети:

![alt text](Net_Scheme_Project.png)

## Таблица адресов:
| Подсеть ipv4 | Device/Port|    Описание   |
|--------------|:----------:| -----------------:|
| 192.168.10.1/32  | Spine-1-1/Lo1 |     Loopback1     |
| 192.168.10.2/32  | Spine-1-2/Lo1 |     Loopback1     |
| 192.168.12.1/32  |  Leaf-1-1/Lo1 |     Loopback1     |
| 192.168.13.1/32  |  Leaf-1-1/Lo2 |     Loopback2     |
| 192.168.12.2/32  |  Leaf-1-2/Lo1 |     Loopback1     |
| 192.168.13.2/32  |  Leaf-1-2/Lo2 |     Loopback2     |
| 192.168.12.3/32  |  Leaf-1-3/Lo1 |     Loopback1     |
| 192.168.13.3/32  |  Leaf-1-3/Lo2 |     Loopback2     |
| 192.168.12.4/32  |  Leaf-1-4/Lo1 |     Loopback1     |
| 192.168.13.4/32  |  Leaf-1-4/Lo2 |     Loopback2     |
| 192.168.14.0/31  |  Spine-1-1 Eth1 |     P2P Spine 1-1 to Leaf 1-1    |
| 192.168.14.1/31  |  Leaf-1-1 Eth1 |     P2P Spine 1-1 to Leaf 1-1    |
| 192.168.14.2/31  |  Spine-1-1 Eth2 |     P2P Spine 1-1 to Leaf 1-2    |
| 192.168.14.3/31  |  Leaf-1-2 Eth1 |     P2P Spine 1-1 to Leaf 1-2    |
| 192.168.14.4/31  |  Spine-1-1 Eth3 |     P2P Spine 1-1 to Leaf 1-3    |
| 192.168.14.5/31  |  Leaf-1-3 Eth1 |     P2P Spine 1-1 to Leaf 1-3    |
| 192.168.14.6/31  |  Spine-1-2 Eth1 |     P2P Spine 1-2 to Leaf 1-1    |
| 192.168.14.7/31  |  Leaf-1-1 Eth2 |     P2P Spine 1-2 to Leaf 1-1    |
| 192.168.14.8/31  |  Spine-1-2 Eth2 |     P2P Spine 1-2 to Leaf 1-2    |
| 192.168.14.9/31  |  Leaf-1-2 Eth2 |     P2P Spine 1-2 to Leaf 1-2    |
| 192.168.14.10/31  |  Spine-1-2 Eth3 |     P2P Spine 1-2 to Leaf 1-3    |
| 192.168.14.11/31  |  Leaf-1-3 Eth2 |     P2P Spine 1-2 to Leaf 1-3    |
| 192.168.14.12/31  |  Spine-1-1 Eth4 |     P2P Spine 1-1 to Leaf 1-4    |
| 192.168.14.13/31  |  Leaf-1-4 Eth1 |     P2P Spine 1-1 to Leaf 1-4    |
| 192.168.14.14/31  |  Spine-1-2 Eth4 |     P2P Spine 1-2 to Leaf 1-4    |
| 192.168.14.15/31  |  Leaf-1-4 Eth2 |     P2P Spine 1-2 to Leaf 1-4    |
| 192.168.20.1/32  | Spine-2-1/Lo1 |     Loopback1     |
| 192.168.20.2/32  | Spine-2-2/Lo1 |     Loopback1     |
| 192.168.22.1/32  |  Leaf-2-1/Lo1 |     Loopback1     |
| 192.168.23.1/32  |  Leaf-2-1/Lo2 |     Loopback2     |
| 192.168.22.2/32  |  Leaf-2-2/Lo1 |     Loopback1     |
| 192.168.23.2/32  |  Leaf-2-2/Lo2 |     Loopback2     |
| 192.168.22.3/32  |  Leaf-2-3/Lo1 |     Loopback1     |
| 192.168.23.3/32  |  Leaf-2-3/Lo2 |     Loopback2     |
| 192.168.22.4/32  |  Leaf-2-4/Lo1 |     Loopback1     |
| 192.168.23.4/32  |  Leaf-2-4/Lo2 |     Loopback2     |
| 192.168.24.0/31  |  Spine-2-1 Eth1 |     P2P Spine 2-1 to Leaf 2-1    |
| 192.168.24.1/31  |  Leaf-2-1 Eth1 |     P2P Spine 2-1 to Leaf 2-1    |
| 192.168.24.2/31  |  Spine-2-1 Eth1 |     P2P Spine 2-2 to Leaf 2-1    |
| 192.168.24.3/31  |  Leaf-2-1 Eth2 |     P2P Spine 2-2 to Leaf 2-1    |
| 192.168.24.4/31  |  Spine-2-1 Eth2 |     P2P Spine 2-1 to Leaf 2-2    |
| 192.168.24.5/31  |  Leaf-2-2 Eth1 |     P2P Spine 2-1 to Leaf 2-2    |
| 192.168.24.6/31  |  Spine-2-2 Eth1 |     P2P Spine 2-2 to Leaf 2-2    |
| 192.168.24.7/31  |  Leaf-2-2 Eth2 |     P2P Spine 2-2 to Leaf 2-2    |
| 192.168.24.8/31  |  Spine-2-1 Eth3 |     P2P Spine 2-1 to Leaf 2-3    |
| 192.168.24.9/31  |  Leaf-2-3 Eth1 |     P2P Spine 2-1 to Leaf 2-3    |
| 192.168.24.10/31  |  Spine-2-2 Eth3 |     P2P Spine 2-2 to Leaf 2-3    |
| 192.168.24.11/31  |  Leaf-2-3 Eth2 |     P2P Spine 2-2 to Leaf 2-3    |
| 192.168.24.12/31  |  Spine-2-1 Eth4 |     P2P Spine 2-1 to Leaf 2-4    |
| 192.168.24.13/31  |  Leaf-2-4 Eth1 |     P2P Spine 2-1 to Leaf 2-4    |
| 192.168.24.14/31  |  Spine-2-2 Eth4 |     P2P Spine 2-2 to Leaf 2-4    |
| 192.168.24.15/31  |  Leaf-2-4 Eth2 |     P2P Spine 2-2 to Leaf 2-4    |
| 172.30.254.0/31  |  Leaf-1-4 Eth3 |     DCI P2P Leaf-1-4 to Leaf 2-1    |
| 172.30.254.1/31  |  Leaf-2-1 Eth3 |     DCI P2P Leaf-1-4 to Leaf 2-1    |
| 172.30.254.2/31  |  Leaf-1-3 Eth3 |     DCI P2P Leaf-1-3 to Leaf 2-2    |
| 172.30.254.3/31  |  Leaf-2-2 Eth3 |     DCI P2P Leaf-1-3 to Leaf 2-2    |

## Настройки коммутаторов:
Использованы шаблоны для ускорения настройки:

SPINE - Underlay BGP на LEAF в сторону SPINE.

LEAFS - Underlay BGP на SPINE в торону LEAF.

OVERLAY - на LEAF и SPINE для настройки EVPN.

BGW - На Border LEAF в сторону LEAF соседнего DC для настройки EVPN.

Для упрощения настройки SPINE использованы комманды bgp listen range. 
Что позволяет уйти от ручного указания IP всех соседствующих LEAF.

### Типовая конфигурация процесса BGP Spine UNDERLAY:
```console
router bgp 65XXXX
   router-id <IP Loopback1>
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   bgp listen range 192.168.X4.0/24 peer-group LEAFS remote-as 65XXXX
   neighbor LEAFS peer group
   neighbor LEAFS next-hop-self
   neighbor LEAFS bfd
   neighbor LEAFS rib-in pre-policy retain all
   neighbor LEAFS route-reflector-client
   neighbor LEAFS password 7 1RuAvIkzlaIS2dTpf+q14g==
   neighbor LEAFS send-community standard extended

   address-family ipv4
      neighbor LEAFS activate
      network <IP Loopback1>

```
### Типовая конфигурация процесса BGP Spine OVERLAY:
```console
service routing protocols model multi-agent

router bgp 65XXXX
   bgp listen range 192.168.X2.0/24 peer-group OVERLAY remote-as 65XXXX
   neighbor OVERLAY peer group
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY route-reflector-client
   neighbor OVERLAY password 123test
   neighbor OVERLAY send-community extended

   address-family evpn
      neighbor OVERLAY activate
   
```
### Типовая конфигурация процесса BGP Leaf UNDERLAY:
```console
router bgp 65XXXX
   router-id <IP Loopback1>
   no bgp default ipv4-unicast
   distance bgp 20 200 200
   maximum-paths 4 ecmp 4
   neighbor SPINE peer group
   neighbor SPINE remote-as 65XXXX
   neighbor SPINE next-hop-self
   neighbor SPINE bfd
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 123test
   neighbor SPINE send-community standard extended

   neighbor <SPINE1 PtP IP> peer group SPINE
   neighbor <SPINE2 PtP IP> peer group SPINE

   address-family ipv4
      neighbor SPINE activate
      network <IP Loopback1>
      network <IP Loopback2>
```

### Типовая конфигурация процесса BGP Leaf OVERLAY:
```console
service routing protocols model multi-agent

router bgp 65XXXX
   neighbor OVERLAY peer group
   neighbor OVERLAY remote-as 65XXXX
   neighbor OVERLAY update-source Loopback1
   neighbor OVERLAY password 123test
   neighbor OVERLAY send-community extended

   neighbor <SPINE1 IP Loopback1> peer group OVERLAY
   neighbor <SPINE2 IP Loopback1> peer group OVERLAY

   address-family evpn
      neighbor OVERLAY activate

```
### Типовая конфигурация процесса BGP Leaf BGW:
```console
service routing protocols model multi-agent

router bgp 65XXXX
   neighbor BGW peer group
   neighbor BGW remote-as 65XXXX
   neighbor BGW password 123test
   neighbor BGW bfd
   neighbor BGW send-community extended

   neighbor <BGW PtP IP> peer group BGW

   address-family evpn
      neighbor BGW activate
      neighbor BGW domain remote

```
### Типовая конфигурация процесса BGP Leaf VXLAN:
```console
interface Vxlan1
   vxlan source-interface Loopback2
   vxlan udp-port 4789
   vxlan vlan <VLAN ID> vni <VNI ID>
   vxlan learn-restrict any

router bgp 65XXXX
  vlan <VLAN ID>
      rd <AS:VNI>
      route-target both <AS:VNI>
      redistribute learned

```
### Типовая конфигурация VRF и SVI под Подразделение:
```console
!
vrf instance <TENANT1>
!
ip routing vrf <TENANT1>
!
interface Vxlan1
   vxlan vrf <TENANT1> vni <номер VNI>
!
router bgp 65XXXX
   vrf <TENANT1>
      rd <AS:VNI>
      route-target import evpn <AS:VNI>
      route-target export evpn <AS:VNI>
      redistribute connected
!
interface Vlan <Номер VLAN>
   description <TENANT1_SERVICE1>
   vrf <TENANT1>
   ip address virtual <IP>

```
### Типовая конфигурация ESI на Port-channel:
```console
!
interface Port-Channel <X>
   evpn ethernet-segment
      identifier XXXX:XXXX:XXXX:XXXX:XXXX
      route-target import XX:XX:XX:XX:XX:XX
   lacp system-id XX:XX:XX:XX:XX:XX
```

### Типовая конфигурация растягивания L2VNI между Датацентрами через DCI MULTI-SITE на BGW:
```console
!
router bgp 65XXXX
   vlan <Номер VLAN>>
      rd <AS:VNI>
      rd evpn domain remote <65500:VNI>
      route-target both <AS:VNI>
      route-target import export evpn domain remote <65500:VNI>
      redistribute learned
!
```

### Типовая конфигурация растягивания L3VNI между Датацентрами через DCI MULTI-SITE на BGW:
```console
!
router bgp 65XXXX
   address-family evpn
      neighbor BGW next-hop-self received-evpn-routes route-type ip-prefix inter-domain
!
```

### Конфиги коммутаторов:
[Все конфиги](/labs/project/configs/)
#### Датацентр1:
[SPINE-1-1](/labs/project/configs/SPINE-1-1_cfg.txt)

[SPINE-1-2](/labs/project/configs/SPINE-1-2_cfg.txt)

[LEAF-1-1](/labs/project/configs/LEAF-1-1_cfg.txt)

[LEAF-1-2](/labs/project/configs/LEAF-1-2_cfg.txt)

[LEAF-1-3](/labs/project/configs/LEAF-1-3_cfg.txt)

[LEAF-1-4](/labs/project/configs/LEAF-1-4_cfg.txt)

#### Датацентр2:
[SPINE-2-1](/labs/project/configs/SPINE-2-1_cfg.txt)

[SPINE-2-2](/labs/project/configs/SPINE-2-2_cfg.txt)

[LEAF-2-1](/labs/project/configs/LEAF-2-1_cfg.txt)

[LEAF-2-2](/labs/project/configs/LEAF-2-2_cfg.txt)

[LEAF-2-3](/labs/project/configs/LEAF-2-3_cfg.txt)

[LEAF-2-4](/labs/project/configs/LEAF-2-4_cfg.txt)

## Вывод комманд маршрутизации

Из-за ограничений по мощности стенда часть оборудования была выключена на момент сбора комманд. Поэтмоу сбор выполнялся с части оборудования, а так же может содержать не все оборудование описанное выше.

### SPINE-1-1:
```console
sh bgp evpn summary 
BGP summary information for VRF default
Router identifier 192.168.10.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.12.1 4 65000             63        74    0    0 00:46:59 Estab   7      7
  192.168.12.2 4 65000            104       121    0    0 01:21:48 Estab   4      4
  192.168.12.3 4 65000            114       106    0    0 01:20:30 Estab   4      4
```
### SPINE-2-1:
```console
sh bgp evp summary 
BGP summary information for VRF default
Router identifier 192.168.20.1, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.22.2 4 65002             11        15    0    0 00:02:52 Estab   3      3
  192.168.22.3 4 65002            109       116    0    0 01:22:21 Estab   8      8
```
### LEAF-1-1:
```console
!
sh vxlan address-table 
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6807  EVPN      Vx1  192.168.13.3     1       0:04:01 ago
  30  0050.7966.6808  EVPN      Vx1  192.168.13.2     1       0:48:41 ago
4093  5000.0003.3766  EVPN      Vx1  192.168.13.2     1       0:48:42 ago
4093  5000.0015.f4e8  EVPN      Vx1  192.168.13.3     1       0:04:02 ago
Total Remote Mac Addresses for this criterion: 4
!
sh bgp evpn
BGP routing table information for VRF default
Router identifier 192.168.12.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65000:100010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 65000:100010 mac-ip 0050.7966.6807
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 mac-ip 0050.7966.6807 10.0.0.12
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 mac-ip 0050.7966.6808
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 mac-ip 0050.7966.6808 10.2.0.13
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 imet 192.168.13.1
                                 -                     -       -       0       i
 * >      RD: 65000:100020 imet 192.168.13.1
                                 -                     -       -       0       i
 * >      RD: 65000:200030 imet 192.168.13.1
                                 -                     -       -       0       i
 * >      RD: 65000:1001 imet 192.168.13.2
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 imet 192.168.13.3
                                 192.168.13.3          -       100     0       i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 ip-prefix 10.0.0.0/24
                                 -                     -       -       0       i
 * >      RD: 65000:1001 ip-prefix 10.0.2.0/24
                                 -                     -       -       0       i
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 -                     -       -       0       i
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1                    
!
sh ip route vrf TENANT1

 C        10.0.0.0/24 is directly connected, Vlan10
 C        10.0.2.0/24 is directly connected, Vlan20
!
sh ip route vrf TENANT2

 B I      10.2.0.13/32 [200/0] via VTEP 192.168.13.2 VNI 1002 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        10.2.0.0/24 is directly connected, Vlan30
 B I      10.2.2.0/24 [200/0] via VTEP 192.168.13.3 VNI 1002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
!
```
### LEAF-1-2:
```console
!
sh vxlan address-table 
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
4093  5000.0015.f4e8  EVPN      Vx1  192.168.13.3     1       0:07:24 ago
4093  5000.00d5.5dc0  EVPN      Vx1  192.168.13.1     1       0:52:08 ago
4094  5000.00d5.5dc0  EVPN      Vx1  192.168.13.1     1       0:52:08 ago
Total Remote Mac Addresses for this criterion: 3
!
sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65000:100010 mac-ip 0050.7966.6806
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 mac-ip 0050.7966.6807
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 mac-ip 0050.7966.6807 10.0.0.12
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65000:1001 mac-ip 0050.7966.6808 10.2.0.13
                                 -                     -       -       0       i
 * >      RD: 65000:100010 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:100020 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:200030 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 imet 192.168.13.2
                                 -                     -       -       0       i
 * >      RD: 65000:100010 imet 192.168.13.3
                                 192.168.13.3          -       100     0       i Or-ID: 192.168.12.3 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 ip-prefix 10.0.0.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 ip-prefix 10.0.2.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 -                     -       -       0       i
 *        RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24
                                 192.168.13.3          -       100     0       65002 i Or-ID: 192.168.12.3 C-LST: 192.168.10.1  
!
sh ip route vrf TENANT1
VRF: TENANT1
Gateway of last resort is not set

!
sh ip route vrf TENANT2
 C        10.2.0.0/24 is directly connected, Vlan30
 B I      10.2.2.0/24 [200/0] via VTEP 192.168.13.3 VNI 1002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
!
```
### LEAF-1-3:
```console
!
sh vxlan address-table 
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  192.168.13.1     1       0:51:57 ago
  10  0050.7966.6807  EVPN      Vx1  192.168.23.2     1       0:10:38 ago
4093  5000.0003.3766  EVPN      Vx1  192.168.13.2     1       1:28:53 ago
4093  5000.00ba.c6f8  EVPN      Vx1  192.168.23.2     1       0:10:38 ago
4093  5000.00d5.5dc0  EVPN      Vx1  192.168.13.1     1       0:55:20 ago
4094  5000.00d5.5dc0  EVPN      Vx1  192.168.13.1     1       0:55:20 ago
Total Remote Mac Addresses for this criterion: 6
!
sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65000:100010 mac-ip 0050.7966.6806
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 mac-ip 0050.7966.6807
                                 -                     -       100     0       65002 i
 * >      RD: 65000:100010 mac-ip 0050.7966.6807 10.0.0.12
                                 -                     -       100     0       65002 i
 * >      RD: 65000:1001 mac-ip 0050.7966.6808
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 mac-ip 0050.7966.6808 10.2.0.13
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65500:10010 mac-ip 0050.7966.6806 remote
                                 -                     -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65500:100010 mac-ip 0050.7966.6807 remote
                                 192.168.23.2          -       100     0       65002 i
 * >      RD: 65500:100010 mac-ip 0050.7966.6807 10.0.0.12 remote
                                 192.168.23.2          -       100     0       65002 i
 * >      RD: 65000:100010 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:100020 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:200030 imet 192.168.13.1
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 imet 192.168.13.2
                                 192.168.13.2          -       100     0       i Or-ID: 192.168.12.2 C-LST: 192.168.10.1 
 * >      RD: 65000:100010 imet 192.168.13.3
                                 -                     -       -       0       i
 * >      RD: 65500:10010 imet 192.168.13.3 remote
                                 -                     -       -       0       i
 * >      RD: 65500:100010 imet 192.168.23.2 remote
                                 192.168.23.2          -       100     0       65002 i
 * >      RD: 65000:1001 ip-prefix 10.0.0.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1001 ip-prefix 10.0.2.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 192.168.13.1          -       100     0       i Or-ID: 192.168.12.1 C-LST: 192.168.10.1 
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24
                                 192.168.23.2          -       100     0       65002 i
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24 remote
                                 192.168.13.1          -       100     0       i
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24 remote
                                 192.168.23.2          -       100     0       65002 i
!
sh ip route vrf TENANT1
Gateway of last resort is not set

!
sh ip route vrf TENANT2
Gateway of last resort is not set

 B I      10.2.0.13/32 [200/0] via VTEP 192.168.13.2 VNI 1002 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B I      10.2.0.0/24 [200/0] via VTEP 192.168.13.1 VNI 1002 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      10.2.2.0/24 [20/0] via VTEP 192.168.23.2 VNI 1002 router-mac 50:00:00:ba:c6:f8 local-interface Vxlan1
!
```

### LEAF-2-2:
```console
!
show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  192.168.13.3     1       0:55:42 ago
  10  0050.7966.6807  EVPN      Vx1  192.168.23.3     1       0:14:24 ago
4093  5000.00d8.ac19  EVPN      Vx1  192.168.23.3     1       0:14:25 ago
4094  5000.0015.f4e8  EVPN      Vx1  192.168.13.3     1       1:16:14 ago
4094  5000.00d8.ac19  EVPN      Vx1  192.168.23.3     1       0:14:25 ago
Total Remote Mac Addresses for this criterion: 5
!
sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65002:100010 mac-ip 0050.7966.6806
                                 -                     -       100     0       65000 i
 * >      RD: 65002:100010 mac-ip 0050.7966.6807
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65002:100010 mac-ip 0050.7966.6807 10.0.0.12
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65002:200040 mac-ip 0050.7966.6809
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65002:200040 mac-ip 0050.7966.6809 10.2.2.14
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65500:10010 mac-ip 0050.7966.6806 remote
                                 192.168.13.3          -       100     0       65000 i
 * >      RD: 65500:100010 mac-ip 0050.7966.6807 remote
                                 -                     -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65500:100010 mac-ip 0050.7966.6807 10.0.0.12 remote
                                 -                     -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65002:100010 imet 192.168.23.2
                                 -                     -       -       0       i
 * >      RD: 65002:100010 imet 192.168.23.3
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65002:200040 imet 192.168.23.3
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65500:10010 imet 192.168.13.3 remote
                                 192.168.13.3          -       100     0       65000 i
 * >      RD: 65500:100010 imet 192.168.23.2 remote
                                 -                     -       -       0       i
 * >      RD: 65002:1001 ip-prefix 10.0.0.0/24
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 192.168.13.3          -       100     0       65000 i
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24
                                 192.168.23.3          -       100     0       i Or-ID: 192.168.22.3 C-LST: 192.168.20.1 
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24 remote
                                 192.168.13.3          -       100     0       65000 i
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24 remote
                                 192.168.23.3          -       100     0       i

!
sh ip route vrf TENANT1
Gateway of last resort is not set

!
sh ip route vrf TENANT2
Gateway of last resort is not set

 B E      10.2.0.0/24 [20/0] via VTEP 192.168.13.3 VNI 1002 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B I      10.2.2.14/32 [200/0] via VTEP 192.168.23.3 VNI 1002 router-mac 50:00:00:d8:ac:19 local-interface Vxlan1
 B I      10.2.2.0/24 [200/0] via VTEP 192.168.23.3 VNI 1002 router-mac 50:00:00:d8:ac:19 local-interface Vxlan1
!
```

### LEAF-2-3:
```console
!
show vxlan address-table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  192.168.23.2     1       0:17:23 ago
4094  5000.00ba.c6f8  EVPN      Vx1  192.168.23.2     1       0:17:23 ago
Total Remote Mac Addresses for this criterion: 2
!
sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65002:100010 mac-ip 0050.7966.6806
                                 192.168.23.2          -       100     0       65000 i Or-ID: 192.168.22.2 C-LST: 192.168.20.1 
 * >      RD: 65002:100010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 65002:100010 mac-ip 0050.7966.6807 10.0.0.12
                                 -                     -       -       0       i
 * >      RD: 65002:200040 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 65002:200040 mac-ip 0050.7966.6809 10.2.2.14
                                 -                     -       -       0       i
 * >      RD: 65002:100010 imet 192.168.23.2
                                 192.168.23.2          -       100     0       i Or-ID: 192.168.22.2 C-LST: 192.168.20.1 
 * >      RD: 65002:100010 imet 192.168.23.3
                                 -                     -       -       0       i
 * >      RD: 65002:200040 imet 192.168.23.3
                                 -                     -       -       0       i
 * >      RD: 65002:1001 ip-prefix 10.0.0.0/24
                                 -                     -       -       0       i
 * >      RD: 65000:1002 ip-prefix 10.2.0.0/24
                                 192.168.23.2          -       100     0       65000 i Or-ID: 192.168.22.2 C-LST: 192.168.20.1 
 * >      RD: 65002:1002 ip-prefix 10.2.2.0/24
                                 -                     -       -       0       i

!
sh ip route vrf TENANT1
Gateway of last resort is not set

 C        10.0.0.0/24 is directly connected, Vlan10

!
sh ip route vrf TENANT2
Gateway of last resort is not set

 B I      10.2.0.0/24 [200/0] via VTEP 192.168.23.2 VNI 1002 router-mac 50:00:00:ba:c6:f8 local-interface Vxlan1
 C        10.2.2.0/24 is directly connected, Vlan40
!
```

### VPC1:
```console
VPC-1> ping 10.0.0.12

84 bytes from 10.0.0.12 icmp_seq=1 ttl=64 time=432.644 ms
84 bytes from 10.0.0.12 icmp_seq=2 ttl=64 time=171.053 ms
84 bytes from 10.0.0.12 icmp_seq=3 ttl=64 time=152.134 ms
84 bytes from 10.0.0.12 icmp_seq=4 ttl=64 time=132.663 ms
84 bytes from 10.0.0.12 icmp_seq=5 ttl=64 time=326.002 ms

```

### VPC2:
```console
VPC-2> ping 10.0.0.11

84 bytes from 10.0.0.11 icmp_seq=1 ttl=64 time=170.550 ms
84 bytes from 10.0.0.11 icmp_seq=2 ttl=64 time=128.551 ms
84 bytes from 10.0.0.11 icmp_seq=3 ttl=64 time=198.727 ms
84 bytes from 10.0.0.11 icmp_seq=4 ttl=64 time=297.980 ms
84 bytes from 10.0.0.11 icmp_seq=5 ttl=64 time=173.316 ms

```

### VPC3:
```console
VPC-3> ping 10.2.2.14

84 bytes from 10.2.2.14 icmp_seq=1 ttl=60 time=117.374 ms
84 bytes from 10.2.2.14 icmp_seq=2 ttl=60 time=190.646 ms
84 bytes from 10.2.2.14 icmp_seq=3 ttl=60 time=148.799 ms
84 bytes from 10.2.2.14 icmp_seq=4 ttl=60 time=155.076 ms
84 bytes from 10.2.2.14 icmp_seq=5 ttl=60 time=185.470 ms

```
### VPC4:
```console
VPC-4> ping 10.2.0.13   

84 bytes from 10.2.0.13 icmp_seq=1 ttl=60 time=181.694 ms
84 bytes from 10.2.0.13 icmp_seq=2 ttl=60 time=208.278 ms
84 bytes from 10.2.0.13 icmp_seq=3 ttl=60 time=160.420 ms
84 bytes from 10.2.0.13 icmp_seq=4 ttl=60 time=153.657 ms
84 bytes from 10.2.0.13 icmp_seq=5 ttl=60 time=120.134 ms

```