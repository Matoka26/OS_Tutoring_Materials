# Task Scheduling
## 1. Motivatie
- Programele trebuie aduse de pe ***disk*** in memorie si pornit un proces pentru a executa
- Vrem sa executam operatiile corect pentru a **proteja** memoria

## 2. Ierarhia memoriei
1. Registri
- se afla **in CPU**
- au cel mai rapid acces, **sub 1CC**
- sunt **mici**, doar cativa bytes

2. Cache
- se afla *aproape de CPU*
- unitati mici de memorie
- acces rapid
- se pun datele **accesate frecvent(temporal)** sau **in localitate(spatial)**, mai multe [aici](https://www.geeksforgeeks.org/locality-of-reference-and-cache-operation-in-cache-memory/)
    - explicat informal: Te apuci sa faci curat in casa. Iti iei o carpa si un spray de mobila din debara si incepi sa stergi praful, iti iei dupa tine si pamatuful (erau pe acelasi raft) pentru ca te astepti sa fie nevoie si de el si ar fi neplacut sa te duci iar dupa el, daca il folosesti ai un ***cache hit***. Daca se intampla sa fie asa dezastru in casa in cat sa ai nevoie si de trafalet sa varuiesti iar... ai un ***cache miss***, nu aveai obiectul la indemana

3. Memoria Principala
- mare
- lenta
- tine date si instructiuni de care are nevoie in prezent CPU-ul
- **volatila**, cand stingi PC-ul dispar (analog pentru celelalte nivele de mai sus)

4. Memoria Secundata
-  ***hard disk drives*** (HDD) si ***solid-state drives*** (SSD)
- **non-volatila**
- tine instructiuni de care **nu are nevoie CPU-ul** in prezent

5. Disk Magnetic
- plate-uri circulare dintr-un material magnetic
- bune la viteze mari
- deseori folosite in PC-uri

6. Benzi magnetice
- un device magnetic de inregistrat acoperit intr-un film de plastic
- foarte lent
- folosit pentru back-up

| **Level**       | **1**            | **2**            | **3**             | **4**               |
|-----------------|------------------|------------------|-------------------|---------------------|
| **Name**        | Register         | Cache            | Main Memory       | Secondary Memory    |
| **Size**        | <1 KB            | less than 16 MB  | <16GB            | >100 GB             |
| **Access Time** | 0.25ns to 0.5ns  | 0.5 to 25ns      | 80ns to 250ns     | 5,000,000 ns          |
| **Bandwidth**   | 20,000 to 100,000 MB | 5,000 to 15,000 MB | 1,000 to 5,000 MB  | 20 to 150 MB        |
| **Managed by**  | Compiler         | Hardware         | Operating System  | Operating System    |
| **Backing Mechanism** | From cache | From Main Memory | From Secondary Memory | From I/O        |

(*)banwidth = marimea canalului prin care se transmit datele

![mem_hierarchy](assets/mem_hierarchy.png)

## 3. Base and Limit
- OS-ul este cel care aloca procesele, el decide cat spatiu poate ocupa fiecare, ***spatiul de adrese***
- Avem nevoie de un mod simplu de adresare, usor de modificat in cazul in care realocam procesul in alt spatiu
- De ce informatii ale unui proces avem nevoie pentru a stii de unde pana unde se afla?
    - ***Base***  -> de unde incepe
    - ***Limit*** -> cat de mare e

![base_limit](assets/base_limit.png)

- Primim o adresa de memorie si vrem sa verificam daca se afla in bound-ul procesului nostru.
- Daca nu se afla in bound, 'atingem' alt proces sau o zona reziduala -> ***segmentation fault***
- un program malitios ar putea incerca sa iasa din bound-ul procesului, sau un program scris prost :)
- Orice proces ce incearca sa iasa din address space-ul lui vi fi **omorat** de OS.
- Check-ul este realizat la fiecare accesare de memorie si este **implementat hardware**

![hardware_base_limit](assets/hardware_base_limit.png)

> [!NOTE]  
> De ce tinem in ***limit*** marimea procesului si nu adresa la care se termina?

## 4. Address Binding
- Daca nu gasim un mecanism de realocare, toate procesele ar astepta sa inceapa de la adresa 0x0000, adica putem rula un singur program la un moment de timp, deci pierdem tot paralelismul

- Care e primul proces de pe un PC si ce implicatii are cu punctul precedent?

- De asemenea, adresele arata diferit de-a lungul unui program:

| **Stage**        | **Address Type**        | **Example**                         |
|------------------|-------------------------|--------------------------------------|
| Source Code      | Symbolic addresses      | `int x`, `goto LABEL`               |
| Compiled Code    | Relocatable addresses   | “14 bytes from the start of module” |
| Execution        | Absolute addresses (physical) | `0x74014` (actual memory location)  |

### Reminder: Trecerea de la Program la Proces
![compilare](assets/compilare.png)

## 5. Spatii de memorie Logice si Fizice
- Adrese logice: 
    - generate de CPU
    - referite ca ***adrese virtuale***
    - "fake"
- Adrese fizice:
    - ***adrese reale*** de din hardware
    - asa cum sunt vazute de Memory-Management Unit (MMU)

- Sunt aceleasi intre ***compile-time*** si ***load-time***
- Difera la ***execution-time***

- Logical address space -> toate adresele logice generate de un proces
- Physical address space -> toate adresele fizice generate de un proces

> [!NOTE]  
> Programul unui user vede numai **adrese logice**, nu stie la ce **adrese fizice** se afla

## 6. Memory-Management Unit (MMU)
- Componenta **hardware**
- Mapeaza la **run-time** adresele ***logice(virtuale)*** de cele ***fizice***
- Exista mai multe implementari
    - la fiecare adresa din user process adaugam valoarea din ***registrul de relocare*** (base-ul de mai devreme)

![mmu_add](assets/mmu_add.png)

## 7. Dynamic Linking
***Linking Static*** 
- Codul din biblioteci esti combinat de loader cu cel din program in imaginea binarului

***Linking Dynamic*** 
- Linking-ul se face la run-time
- Daca o bucata de cod lipseste din address space-ul procesului, este cautata si adaugata
- Foarte util cand folosim librarii
- Se poate ca versiunea metodei din librarie sa fie ***deprecated*** si sa aflam tarziu, deci conteaza versiunile librariilor

## 8. Swapping
- Un proces poate fi **scos** din memoria principala si adus mai tarziu inapoi pentru a continua executia
- ***Backing store*** 
    - disk rapid suficient de mare pentru a tine mai multe copii de procese pentru toti userii
    - trebuie sa ofere acces direct catre imagini
- ***Roll out, roll in***
    - varianta de swapping pentru algorimtii de schedulebazati pe prioritati

- Mare parte din swap este ocupata de ***transferul datelor***, care e proportional cu dimensiunea memoriei care trebuie mutata (se face hardware deci timpul fizic de transfer al unui block e acelasi pentru toata lumea)

- Sistemul mai tine si o coada pentru procesele ready-to-run de pe disk