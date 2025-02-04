---
layout: post
title: Microsegmentazione nelle Reti Virtuali
categories: [Network, Security, Microsegmentation, Vlan ]
---

## Indice

1. Introduzione
2. Interfacce Linux per il Networking Virtuale
3. Open vSwitch, le Modalità Bridge VEPA Isolate e IEEE 802.1Qbg
4. Modalità di Implementazione della Microsegmentazione
   1. Utilizzo di firewall su ogni nodo degli hypervisor
   2. Utilizzo degli strumenti nativi degli hypervisor
   3. Utilizzo delle Private VLAN
   4. Utilizzo di VEPA e Hairpin
5. Tipi di Virtualizzatori e Supporto alla Microsegmentazione
6. Conclusioni

---

## 1. Introduzione

La microsegmentazione è una tecnica avanzata per migliorare la sicurezza delle reti virtualizzate, segmentando il traffico per ridurre la superficie di attacco e prevenire movimenti laterali delle minacce. Tuttavia, nonostante i suoi vantaggi teorici, la sua implementazione pratica presenta ancora molte sfide e incertezze.

Attualmente, non esiste una soluzione universalmente adottata per la microsegmentazione. Tra le opzioni disponibili, lo standard **IEEE 802.1Qbg** rappresenta una strada promettente per fornire un framework standardizzato e scalabile. Tuttavia, la sua adozione da parte dei principali provider è ancora limitata. Solo recentemente Docker ha iniziato a integrarlo, suggerendo un possibile punto di svolta per il settore.

In mancanza di una standardizzazione consolidata, la scelta dell'approccio più efficace rimane complessa. Sebbene l'utilizzo di firewall su ogni nodo e l'adozione di strumenti nativi degli hypervisor siano tecnicamente possibili, essi risultano spesso difficili da scalare e gestire nel lungo periodo. L'implementazione tramite **Private VLAN (PVLAN)** appare attualmente come la soluzione più percorribile, permettendo un controllo del traffico più semplice e gestibile rispetto ad altre strategie più complesse.

Questo documento esplora le diverse metodologie di microsegmentazione, analizzandone i vantaggi e le difficoltà, con l'obiettivo di fornire una panoramica chiara delle possibili scelte implementative e del loro impatto sulle infrastrutture di rete.\
Questo primo documento servirà poi per valutare le difficoltà di fondo nell'adozione delle varie metodologie

## 2. Interfacce Linux per il Networking Virtuale

Le interfacce di rete in Linux sono componenti fondamentali per la gestione della connettività nei sistemi virtualizzati. Alcune delle principali tipologie includono:

- **veth (Virtual Ethernet Pair):** Un'interfaccia a coppie che collega due namespace di rete Linux, consentendo la comunicazione tra container o VM.
- **bridge:** Simula il comportamento di uno switch di livello 2, consentendo la connessione tra più interfacce di rete.
- **macvlan:** Permette a più container o VM di condividere la stessa interfaccia fisica, ciascuna con un proprio indirizzo MAC.
- **tap/tun:** Interfacce virtuali utilizzate per la creazione di VPN o per l'instradamento del traffico di rete tra VM.

Queste interfacce sono spesso utilizzate in combinazione con Open vSwitch o con configurazioni di microsegmentazione per garantire un controllo più raffinato del traffico di rete.

## 3. Open vSwitch, le Modalità Bridge VEPA Isolate e IEEE 802.1Qbg

Open vSwitch (OVS) è un software switch virtuale open-source progettato per essere utilizzato in ambienti di virtualizzazione e cloud. Fornisce funzionalità avanzate come il controllo del traffico, l'inoltro dinamico dei pacchetti e il supporto per diverse tecnologie di microsegmentazione.

Le modalità bridge VEPA (Virtual Ethernet Port Aggregator) e isolate sono due approcci specifici per gestire il traffico tra macchine virtuali:

- **Bridge VEPA:** Consente di inoltrare tutto il traffico tra le VM a uno switch fisico esterno, consentendo a dispositivi di rete fisici di applicare policy di sicurezza e monitoraggio avanzato.
- **Modalità isolate:** Impedisce alle macchine virtuali di comunicare direttamente tra loro sulla stessa VLAN, costringendo il traffico a passare attraverso dispositivi di rete specifici, come firewall o router.

IEEE 802.1Qbg è una tecnologia emergente che standardizza il comportamento delle modalità VEPA e bridge isolate, consentendo una gestione più efficace del traffico tra macchine virtuali. Questa norma permette di indirizzare il traffico di rete in modo più sicuro ed efficiente, migliorando la scalabilità e la gestione della sicurezza in ambienti virtualizzati.

## 4. Modalità di Implementazione della Microsegmentazione

La microsegmentazione può essere implementata attraverso diverse strategie che permettono di controllare il traffico di rete in ambienti virtualizzati. Ogni approccio presenta specifiche caratteristiche, vantaggi e svantaggi, rendendolo più o meno adatto a seconda delle esigenze di sicurezza e scalabilità dell'infrastruttura.

Di seguito vengono analizzate le quattro principali metodologie di implementazione della microsegmentazione, ciascuna con il proprio funzionamento e impatto sulla gestione delle risorse di rete.

### 4.1 Utilizzo di firewall su ogni nodo degli hypervisor

Questa modalità sfrutta l'uso di dispositivi **VETH (Virtual Ethernet Pair)** per creare tunnel Ethernet locali tra le macchine virtuali e i firewall virtuali su ogni nodo. I dispositivi VETH sono creati in coppia, permettendo il trasferimento immediato di pacchetti tra le due estremità del tunnel.

- **Vantaggi:**

  - Controllo di sicurezza altamente granulare.
  - Protezione efficace contro movimenti laterali di minacce.

- **Svantaggi:**

  - Necessità di rimuovere i vSwitch distribuiti.
  - Necessità di aggiungere 2 virtual firewall per nodo per garantire alta affidabilità.
  - Comporta un notevole incremento delle risorse necessarie.

### 4.2 Utilizzo degli strumenti nativi degli hypervisor

Gli strumenti nativi degli hypervisor offrono soluzioni integrate per la microsegmentazione e la gestione del traffico di rete tra le macchine virtuali. Questi strumenti, come Nutanix Flow per Nutanix AHV o NSX per VMware, consentono di creare policy di sicurezza direttamente nell'infrastruttura di virtualizzazione.

- **Vantaggi:**

  - Integrazione nativa con l’hypervisor.
  - Gestione centralizzata e semplificata.

- **Svantaggi:**

  - Lock-in molto difficile da eliminare.
  - Necessità di formazione specifica e difficile integrazione con altri dispositivi di rete.

### 4.3 Utilizzo delle Private VLAN

Le **Private VLAN (PVLAN)** permettono di isolare macchine virtuali all'interno della stessa VLAN, impedendo la comunicazione diretta tra di loro.

- **Vantaggi:**

  - Isolamento efficace tra le VM senza necessità di firewall per ogni nodo.
  - Riduzione del rischio di attacchi interni tra macchine virtuali.

- **Svantaggi:**

  - Necessità di supporto dagli switch virtuali nel virtualizzatore.
  - Richiede un'estrema conoscenza della rete e dei requisiti delle macchine in ogni VLAN.

### 4.4 Utilizzo di VEPA e Hairpin

Il traffico di rete tra le VM può essere forzato a passare attraverso un firewall fisico esterno utilizzando le tecniche **VEPA (Virtual Ethernet Port Aggregator)** e **Hairpin**.

- **Vantaggi:**

  - Analisi del traffico più approfondita grazie all'uso di firewall fisici.
  - Possibilità di applicare regole di sicurezza avanzate su dispositivi specializzati.

- **Svantaggi:**

  - Aumento considerevole del traffico all’esterno del virtualizzatore.
  - Necessità di supporto dagli switch virtuali nel virtualizzatore.
  - Necessità di supporto dagli switch fisici.

## 5. Tipi di Virtualizzatori e Supporto alla Microsegmentazione

| Tipo di Virtualizzatore | VEPA/Hairpin  | Private VLAN | Microsegmentazione | Firewall Virtuali |
| ----------------------- | ------------- | ------------ | ------------------ | ----------------- |
| ESXi                    | No            | Sì           | N/D                | Sì\*              |
| Nutanix AHV             | No\*\* \*\*\* | No           | Nutanix Flow       | Sì\*              |
| Proxmox                 | No\*\*        | Sì           | -                  | Sì\*              |
| KVM                     | No\*\*        | N/D          | -                  | Sì\*              |
| Hyper-V                 | N/D           | Sì           | N/D                | N/D               |

(\* = Possibile tuttavia è un operazione da configurare da cli rendendo complicata la gestione anche al crearsi di una nuova vm, potrebbero incorrere difficoltà aggiuntive nella migrazione tra nodi del cluster)

(\*\* = Open Virtual Switch ad oggi non supporta ancora VEPA, ma sembra che stiano valutando e discutendo le modalità di implementazione)

(\*\*\* = Anche se Nutanix AHV utilizza Open Virtual Switch, si pensa che non andrà ad implementarlo come per le private VLAN per favorire la sua soluzione proprietaria licenziata Nutanix Flow)

## 6. Conclusioni

La microsegmentazione è un concetto complesso e in costante evoluzione, con numerose sfide implementative e difficoltà di scalabilità. Sebbene offra un alto livello di sicurezza e segmentazione della rete, la sua adozione richiede competenze avanzate e infrastrutture adeguate.

Tra le possibili strategie, l'implementazione tramite **IEEE 802.1Qbg** rappresenta un'opzione promettente per fornire un framework standardizzato e più scalabile. Tuttavia, la sua diffusione è ancora limitata e, al momento, nessun grande player lo ha implementato completamente. Recentemente, Docker ha iniziato a integrarlo, segnando un possibile punto di svolta per l'adozione futura della tecnologia.

Nonostante i progressi, la microsegmentazione rimane una sfida aperta per molte aziende, che devono trovare il giusto equilibrio tra sicurezza, scalabilità e compatibilità con le infrastrutture esistenti. Attualmente, la soluzione più percorribile è l'uso delle **Private VLAN (PVLAN)**, che permette di segmentare la rete con un'implementazione più gestibile.

L'adozione di firewall su ogni nodo o l'impiego di strumenti nativi degli hypervisor, pur essendo tecnicamente fattibile, rappresenta una scelta complessa e poco lungimirante. Queste soluzioni possono risultare difficili da scalare e gestire nel lungo periodo, aumentando la complessità operativa senza garantire un'efficace interoperabilità tra le diverse piattaforme.

Nel futuro, l'industria potrebbe convergere verso un'implementazione più ampia di standard aperti come **IEEE 802.1Qbg**, ma, per ora, il panorama della microsegmentazione rimane frammentato e in fase di sperimentazione.

## 7. Bibliografia

### Docker

1. [Docker Macvlan Network Driver](https://docs.docker.com/engine/network/drivers/macvlan/)

### Nutanix

2. [Nutanix Flow Documentation](https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2094-Flow\:TN-2094-Flow)

### Hyper-V

3. [Hyper-V Private VLANs](https://learn.microsoft.com/en-us/answers/questions/1240162/hyper-v-private-vlans)

### Linux Drivers e Virtualizzazione

4. [Introduction to Linux Interfaces for Virtual Networking](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking)
5. [Virtual Switches Overview](http://wpage.unina.it/rcanonic/didattica/cdn/lucidi/CDN-L06-a-v03-VirtualSwitches.pdf)
6. [Gist: Virtual Networking Setup](https://gist.github.com/nerdalert/3d2b891d41e0fa8d688c)

### IEEE 802.1Qbg e VEPA

7. [IEEE 802.1Qbg VEPA Seminar](https://www.ieee802.org/1/files/public/docs2009/new-hudson-vepa_seminar-20090514d.pdf)
8. [IEEE 802.1Qbg Standard](https://standards.ieee.org/ieee/802.1Qbg/4757/)
9. [IBM Virtual Server 802.1Qbg Profiles](https://www.ibm.com/docs/en/fsmmn?topic=servers-creating-virtual-server-8021-qbg-profiles)
10. [VEPA: An Answer to Virtual Switching](https://www.networkworld.com/article/717441/vepa-an-answer-to-virtual-switching.html)
11. [Edge Virtual Bridging (EVB) 802.1Qbg](https://blog.ipspace.net/2011/05/edge-virtual-bridging-evb-8021qbg-eases/)

### Private VLAN (PVLAN)

12. [Private VLAN (PVLAN) on vNetwork Distributed Switch](https://knowledge.broadcom.com/external/article/311718/private-vlan-pvlan-on-vnetwork-distribut.html)
13. [Private VLANs Trunk Mode for Isolation in Proxmox](https://forum.proxmox.com/threads/private-vlans-trunk-mode-for-isolation-proxmox-ve-vm.110618/)

###