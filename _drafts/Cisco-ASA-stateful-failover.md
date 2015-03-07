---
title: Подводные камни Cisco ASA stateful failover
layout: post
published: false
---

Недавно внедрил на работе [отказоустойчивую пару Cisco ASA 5512-X](http://www.cisco.com/c/en/us/td/docs/security/asa/asa93/configuration/general/asa-general-cli/ha-failover.html#pgfId-1494550). С ASA мы знакомы не первый год, но настраивать HA с ее помощью мне пришлось впервые. Отличительная особенность HA у ASA - так называемый [stateful failover](http://www.cisco.com/c/en/us/td/docs/security/asa/asa93/configuration/general/asa-general-cli/ha-failover.html#pgfId-1495008). Технология очень крутая. Как говорится, "ни единого разрыва". Одно из устройств можно спокойно выключить, перезагрузить, взровать - ни одно соединение не порвется. Достигается такой эффект за счет непрерывной репликации статуса соединений между устройствами по отдельному линку. Реплицируется почти все - arp таблица, tcp сессии, udp сессии, Site-to-Site IPsec (всего около 20 пунктов). 

Документация говорит, что для stateful falover между устройствами необходимо настроить двасоединения - failover link и stateful link. Если есть взможность, рекомендуется пустить эти линки по разным физическим каналам. Если же разных L2 каналов нет, то failover link и stateful link можно объединить. Далее все примеры в документации приводятся для случая, когда мы настраиваем два разных линка. 
```
!
failover lan unit primary
!
failover lan interface folink gigabitethernet0/3
failover interface ip folink 172.27.48.1 255.255.255.0 standby 172.27.48.2
!
interface gigabitethernet 0/3
  no shutdown
!  
failover link statelink gigabitethernet0/4
failover interface ip statelink 172.27.49.1 255.255.255.0 standby 172.27.49.2
!
interface gigabitethernet 0/4
  no shutdown
!
```
В моем случае двух разных физических сред не было. При таком раскладе надежнее настроить один EtherChannel на два разных свитча в стеке, чем городить отдельные линки для каждого соединения. Тут и возникли небольшие проблемы.

### Cisco ASA и Etherchannel

