# Mønstre

## Fagsystem har delegert rettigheter i Virksomhetslommeboken til Kunden.
* Hvordan/Hva/omfang av delegering ?
### 0:  Bevis lommebok-til-lommebok

Ingen behov for Maskinporten.

```mermaid
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW1del[VL- delegated access]
  BW1[Virksomhetslommebok]
end


subgraph TE [Tjenesteeier]
  BW2[Virksomhetslommebok]
  F[Fagsystem]
end

subgraph EU [EU-kommisjonen]
  EDD[European Digital Directory]
end

    
F -->|1. 'send noe til noen'| BW1del
BW1del -->|2.Tilgang OK| BW1
BW1 -->|2. 'finn lommebok-adresse'| EDD
BW1 --->|3. presenter bevis|BW2
```

* Utfordring i automatisk mottak av bevis.
* Sporing av formål / info om hvorfor noe blir sendt. 


### 1:  Data over QRDS lommebok-til-lommebok

Ingen behov for Maskinporten.

```mermaid
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW1del[VL- delegated access]
  BW1[Virksomhetslommebok]
end

subgraph Q [QTSP]
    QRDS[QRDS] 
end

subgraph TE [Tjenesteeier]
  BW2[Virksomhetslommebok]
  F[Fagsystem]
end

subgraph EU [EU-kommisjonen]
  EDD[European Digital Directory]
end

    
F -->|1. 'send noe til noen'| BW1del
BW1del -->|2.Tilgang OK| BW1
BW1 -->|2.QRDS| QRDS
QRDS -->|3.'finn lommebok-adresse'| EDD
QRDS --->|4.Overfør melding|BW2
```
* Strukturert/ Ustrukturert data
* QRDS nasjonal struktur om mulig? -> Kostnadsdriver ?  

### 2. EBWOID erstatter virksomhetssertifikat
Tjenesteeier har et eksisterende API som er sikret med Maskinporten, ønsker å kunne dele også med "virksomhetslommebok".

```mermaid
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW[Virksomhetslommebok]
end

subgraph Felles [Fellesløsninger]
  MP[Maskinporten]
  SA[(tilganger)]
end

subgraph TE [Tjenesteeier]
  A[API]
end

TE -->|konfigurerer|SA
F -->|1.ber om token| BW
BW -->|2. presenterer ebwoid| MP
MP -->|3. sjekker|SA
F ----->|3. sender request til|A
```

Virksomhetslommeboken får her beskjed fra et internt fagsystem om at den må skaffe et maskinporten-token med et bestemt oauth2-scope. 
Vanlig maskinporten-flyt, men istedenfor virksomhetssertifikat brukes EBWOID. Enten presentert eller for å signere JWTen.
utfordringer:
- Presentasjon av bevis er tyngre enn sjekk av sertifikat. I det hele tatt egnet? 
- Maskinporten som tilgangssertifikat-utsteder basert på EBWOID-presentasjon?

### 3. EBWOID + Scope bevis ?

```mermaid
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW[Virksomhetslommebok]
end

subgraph Felles [Fellesløsninger]
  MP[Maskinporten]
  SA[(tilganger)]
end

subgraph TE [Tjenesteeier]
  A[API]
end

TE -->|konfigurerer|SA
F -->|1.ber om token| BW
BW -->|2. presenterer ebwoid + scope bevis| MP
MP -->|3. sjekker|SA
F ----->|3. sender request til|A
```
Gir det mening å forenkle helt? 

```mermaid
graph LR

subgraph TE [Tjenesteeier]
  A[API]
end

subgraph C [Konsument]
  F[Fagsystem]
  BW[Virksomhetslommebok]
end

F -->|1.ber presentation link| BW
BW -->|2. presentation link + data payload| A
A -->|3.henter/validerer bevis|BW
```




- hvilke bevis er det som mapper til tjenesteeiers scope?
    - Spesialiserte scope bevis for en spesifikk tjeneste? 
    - Gjenbruk av bevis - eks. Registrert finans-institusjon? -> Automatisk tilgang
- skal scope-tilganger fortsatt ligge i Maskinporten?
- Maskinporten skrive tilgangsbevis tilbake til virksomhetslommeboken? og så brukes dette direkte mot API ? 




## Fagsystem har egen lommebok, og fullmakter fra Kunde
### 0:  Generelt mot maskinporten: 
```mermaid
graph LR
    subgraph C [Konsument/Fagsystem]
        FVL[Fagsystem Lommebok]
        F[Fagsystem]
    end
    subgraph TE [Tjenesteeier]
        API
    end
    subgraph K [Kunde]
        KVL[Kunde Lommebok]
    end
    subgraph Felles [Fellesløsninger]
        MP[Maskinporten]
        SA[(tilganger)]
    end


    KVL -->|1. Kunde gir PoA| FVL
    F -->|1.ber om token| FVL
    FVL -->|2. presenterer ebwoid + scope + on_behalf_of | MP
    MP -->|3. sjekker|SA
    F ----->|3. sender request til|API
```
* Et fagsystem kan ikke lenger påstå å representere noen, de må faktisk bevise det gjennom en fullmakt. 
* Scope kan byttes ut med et bevis som impliserer scope

### 1: Fagsystem direkte mot API tilbyder
Vi ønsker et mønster der Fagsystemet bruker Bevis/EBWOID til å bevise sin identitet og rettigheter til APIet, samt å overføre dataen til APIet. 


```mermaid
graph LR
    subgraph TE [Tjenesteeier]
        A[API]
    end
subgraph K [Kunde]
  VL[Lommebok]
end
subgraph C [Konsument]
  F[Fagsystem]
  BW[Virksomhetslommebok]
end
VL -->|1. Kunde gir PoA| BW
F -->|1.ber presentation link for nødvendige bevis| BW
F -->|2. presentation link + data payload| A
A -->|3.henter/validerer bevis|BW
```
* Hvordan gjøre presentation link brukbar ? Handoff i OpenID4VCP prosessen?
* Legge ved Bevis istedenfor direkte til APIet? 
* Trigge API-tjenestetilbyder til å generere en credential request først? 