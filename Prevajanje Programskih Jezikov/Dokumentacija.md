# Jezik TriWave - Popolna dokumentacija

**Ime skupine:** TriWave  
**Člani:** Anđela Radaković, Aleksandra Mickoska, David Novak

---

## Kazalo

1. [Uvod](#uvod)
2. [Konstrukti jezika](#konstrukti-jezika)
   - 2.1. [Osnovni konstrukti](#osnovni-konstrukti)
     - 2.1.1. [Enota](#enota)
     - 2.1.2. [Realna števila](#realna-števila)
     - 2.1.3. [Nizi](#nizi)
     - 2.1.4. [Koordinate](#koordinate)
     - 2.1.5. [Bloki](#bloki)
     - 2.1.6. [Ukazi](#ukazi)
   - 2.2. [Ostali konstrukti](#ostali-konstrukti)
     - 2.2.1. [Deklaracije](#deklaracije)
     - 2.2.2. [Izrazi](#izrazi)
     - 2.2.3. [Kontrola toka](#kontrola-toka)
     - 2.2.4. [Procedure](#procedure)
     - 2.2.5. [Metapodatki](#metapodatki)
     - 2.2.6. [Povpraševanja](#povpraševanja)
     - 2.2.7. [Transformacije](#transformacije)
3. [Gramatika](#gramatika)
4. [First in Follow množice](#first-in-follow-množice)
   - 4.1. [First množice](#first-množice)
   - 4.2. [Follow množice](#follow-množice)
5. [Testni primeri](#testni-primeri)

---

## Uvod

Jezik TriWave je domensko specifičen programski jezik, namenjen opisovanju in modeliranju mest. Omogoča definiranje različnih mestnih elementov kot so ceste, zgradbe, parki, reke, mostovi, restavracije, radijske postaje in seznami predvajanja.

---

## Konstrukti jezika

### Osnovni konstrukti

#### Enota

Služi kot prazen element, ki se uporablja za označevanje konca seznama ali odsotnosti vrednosti.

```
nil
```

#### Realna števila

Uporabljajo se za koordinate, kote, dimenzije, frekvence itd.

```
5
3.14
102.34223
```

#### Nizi

Uporabljajo se za imena mest, cest, pesmi, radijskih postaj itd.

```
"STRING"
```

#### Koordinate

Točke v prostoru (zemljepisna dolžina, zemljepisna širina). Prva komponenta je X (dolžina), druga Y (širina).

```
(X, Y)
```

#### Bloki

##### Blok city

Blok **city** predstavlja mesto, ki ga opisujemo. Določen je z imenom mesta. Lahko vsebuje bloke za opis cest, zgradb, radijskih postaj, seznamov predvajanja, parkov, rek ipd. Je glavni element v jeziku.

**Sintaksa:**
```
city "NAME" {
    BLOKI
}
```

**Primer:**
```
city "Maribor" {
    // ostali_bloki
}
```

##### Blok road

Blok **road** predstavlja cesto. Določen je z imenom ceste. Lahko vsebuje ukaze za izris, ki izrišejo črte.

**Sintaksa:**
```
road "NAME" {
    UKAZI
}
```

**Primer:**
```
road "Bulevar kralja Aleksandra" {
    line((0,0),(5,0));
}
```

##### Blok building

Blok **building** predstavlja zgradbo. Določen je z imenom zgradbe. Lahko vsebuje ukaze za izris, ki izrišejo obrobo zapolnjenega lika. Obroba lika mora biti zaključena, torej končna lokacija mora biti enaka prvi.

**Sintaksa:**
```
building "NAME" {
    UKAZI
}
```

**Primer:**
```
building "Občina" {
    box((1,2),(3,1));
}
```

##### Blok radio

Blok **radio** predstavlja radijsko postajo. Določen je z imenom radijske postaje in koordinatami. Vsebuje metapodatke o tej radijski postaji.

**Sintaksa:**
```
radio "NAME" (X, Y) {
    METAPODATKI
}
```

**Primer:**
```
radio "HIT FM" (4.5, 2.0) {
    set("frequency", 101.1);
    set("genre", "rock");
}
```

##### Blok playlist

Blok **playlist** predstavlja seznam predvajanja. Določen je z imenom in koordinatami. Vsebuje metapodatke o tem seznamu predvajanja.

**Sintaksa:**
```
playlist "NAME" (X, Y) {
    METAPODATKI
}
```

**Primer:**
```
playlist "Priljubljeni" (5.2, 8.4) {
    add("Nisi ti za male stvari", "Željko Samardžić");
    add("Noč pod zvezdami", "Lexington");
    set("mood", "prebujanje");
}
```

##### Blok river

Blok **river** predstavlja reko. Določen je z imenom. Lahko vsebuje ukaze za izris, ki izrišejo črte.

**Sintaksa:**
```
river "NAME" {
    UKAZI
}
```

**Primer:**
```
river "Ibar" {
    line((0,0),(5,0));
}
```

##### Blok park

Blok **park** predstavlja park. Določen je z imenom. Lahko vsebuje ukaze za izris, ki izrišejo obrobo zapolnjenega lika. Obroba lika mora biti zaključena, torej končna lokacija mora biti enaka prvi.

**Sintaksa:**
```
park "NAME" {
    UKAZI
}
```

**Primer:**
```
park "Mestni park" {
    box((5,6),(2,1));
}
```

##### Blok bridge

Blok **bridge** predstavlja most. Določen je z imenom. Lahko vsebuje ukaze za izris, ki izrišejo črte.

**Sintaksa:**
```
bridge "NAME" {
    UKAZI
}
```

**Primer:**
```
bridge "Ibarski most" {
    line((0,4),(5,0));
}
```

##### Blok restaurant

Blok **restaurant** predstavlja restavracijo. Določen je z imenom in koordinatami.

**Sintaksa:**
```
restaurant "NAME" (X, Y);
```

**Primer:**
```
restaurant "Mali Leskovac" (6.4, 8.9);
```

#### Ukazi

##### Ukaz line

Ukaz **line** nariše daljico med dvema točkama.

**Sintaksa:**
```
line(TOČKA, TOČKA)
```

**Primer:**
```
line((1.5,2.0), (4.0, 2.5))
```

##### Ukaz bend

Ukaz **bend** nariše krivuljo med dvema točkama z danim kotom. Če je kot 0° se izriše ravna črta, če je kot 45° se izriše približek četrtine krožnice. Pozitivni koti pomenijo, da je krivulja nagnjena v levo, negativni koti pa pomenijo, da je krivulja nagnjena v desno.

**Sintaksa:**
```
bend(TOČKA, TOČKA, KOT)
```

**Primer:**
```
bend((2, 2), (3, 3), 45)
```

##### Ukaz box

Ukaz **box** izriše pravokotnik med podanima lokacijama. Prva lokacija je zgornji levi kot pravokotnika, druga lokacija pa je spodnji desni kot.

**Sintaksa:**
```
box(TOČKA, TOČKA)
```

**Primer:**
```
box((1, 1), (3, 2))
```

##### Ukaz circ

Ukaz **circ** izriše krog z izbranim polmerom na podani lokaciji. Ta lokacija predstavlja center kroga.

**Sintaksa:**
```
circ(TOČKA, POLMER)
```

**Primer:**
```
circ((5, 5), 1.2)
```

### Ostali konstrukti

#### Deklaracije

Deklaracije se uporabljajo za definiranje spremenljivk, konstant, točk in izrazov, ki jih je mogoče uporabiti kasneje.

Spremenljivke lahko vsebujejo: številke, koordinate, izraze...

**Primer:**
```
let p = (2, 2);
let polmer = 1.5;
line(p, (4, 4));
circ(p, polmer);
```

#### Izrazi

- **fst(p)** -- vrne prvo komponento točke p
- **snd(p)** -- vrne drugo komponento točke p
- **+, -, *, /** -- standardne aritmetične operacije
- **lambda(x) { ... }** -- anonimna funkcija

**Primer:**
```
let s = (2, 3);
let p = (fst(s) + 1, snd(s) * 2); // => (3, 6)
```

**Primer lambda funkcije:**
```
let kvadrat = lambda(x) { x * x };
let vrednost = kvadrat(3); // 9
```

#### Kontrola toka

##### If ... else

Pogojno vejanje, odvisno od izraza.

**Primer:**
```
if n > 1 {
    line((1, 1), (2, 2));
} else {
    line((2, 1), (1, 2));
}
```

##### For zanka

Zanka, ki izvede blok for i od a do b (vključno).

**Primer:**
```
for i = 1 to 3 {
    box((i, 0), (i + 1, 1));
}
```

##### Foreach zanka

Iteracija skozi elemente seznama.

**Primer:**
```
let seznam = [1, 2, 3];
foreach x in seznam {
    circ((x, 0), 0.5);
}
```

#### Procedure

Uporabljajo se za abstrahiranje ponovljivih vzorcev, kot je risanje ponavljajočih se elementov. Definira imenovano funkcijo s parametrom.

**Primer:**
```
procedure nadstropje(p) {
    box(p, (fst(p)+2, snd(p)+1));
}
```

#### Metapodatki

Omogoča dodajanje poljubnih lastnosti kateremu koli objektu.

**Primer:**
```
radio "B92" (2, 2) {
    set("frequency", 92.5);
    set("genre", "talk");
}
```

#### Povpraševanja

Uporabljajo se za prostorske poizvedbe, kot je iskanje bližnjih elementov.

**Primer:**
```
let r = neigh((5, 5), 2); // elementi v polmeru 2
foreach x in r {
    highlight x;
}
```

#### Transformacije

Spremeni referenčni koordinatni sistem -- uporablja se za relativno pozicioniranje.

**Primer:**
```
translate (10, 5);
building "Relativna zgradba" {
    box((0, 0), (2, 2));
}
```

---

## Gramatika

```
Program ::= CityDecl

CityDecl ::= city STRING { CityElementList }

CityElementList ::= CityElement CityElementList | Ɛ

CityElement ::= RoadDecl | BuildingDecl | RadioDecl | PlaylistDecl |
                ParkDecl | RiverDecl | BridgeDecl | RestaurantDecl | LetDecl |
                Statement

Statement ::= LetDecl | Assignment | ProcDec | IfStatement |
              ForStatement | ForeachStatement | Expr

RoadDecl ::= road STRING { CommandList }

BuildingDecl ::= building STRING { CommandList }

RadioDecl ::= radio STRING Point { MetadataList }

PlaylistDecl ::= playlist STRING Point { PlaylistBody }

ParkDecl ::= park STRING { CommandList }

RiverDecl ::= river STRING { CommandList }

BridgeDecl ::= bridge STRING { CommandList }

RestaurantDecl ::= restaurant STRING Point

CommandList ::= Command CommandList | Ɛ

Command ::= line (Point, Point); | bend (Point, Point, Expr); | box
            (Point, Point); | circ (Point, Expr);

PlaylistBody ::= PlaylistEntry PlaylistBody | Ɛ

PlaylistEntry ::= add (STRING, STRING); | SetCommand

MetadataList ::= SetCommand MetadataList | Ɛ

SetCommand ::= set (STRING, Expr);

LetDecl ::= let ID = Expr;

Assignment ::= ID = Expr;

Point ::= (Expr, Expr)

Expr ::= Term Expr'

Expr' ::= + Term Expr' | - Term Expr' | Ɛ

Term ::= Factor Term'

Term' ::= * Factor Term' | / Factor Term' | Ɛ

Factor ::= NUMBER | STRING | ID | fst (Expr) | snd (Expr) | (Expr) |
           lambda (ID) {Expr}

IfStatement ::= if Expr {StatementList } ElsePart

ElsePart ::= else { StatementList} | Ɛ

ForStatement ::= for ID = Expr to Expr {StatementList}

ForeachStatement ::= foreach ID in ID {StatementList}

ProcDecl ::= procedure ID (ID) {StatementList}
```

---

## First in Follow množice

### First množice

```
First(Program) = { city }
First(CityDecl) = { city }
First(CityElementList) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, Ɛ }
First(CityElement) = { bridge, building, for, if, let, park, playlist, radio, restaurant, river, road }
First(StatementList) = { ID, for, if, let, Ɛ }
First(Statement) = { ID, for, if, let }
First(RoadDecl) = { road }
First(BuildingDecl) = { building }
First(RadioDecl) = { radio }
First(PlaylistDecl) = { playlist }
First(ParkDecl) = { park }
First(RiverDecl) = { river }
First(BridgeDecl) = { bridge }
First(RestaurantDecl) = { restaurant }
First(CommandList) = { bend, box, circ, line, Ɛ }
First(Command) = { bend, box, circ, line }
First(PlaylistBody) = { add, set, Ɛ }
First(PlaylistEntry) = { add, set }
First(MetadataList) = { set, Ɛ }
First(SetCommand) = { set }
First(LetDecl) = { let }
First(Assignment) = { ID }
First(Point) = { ( }
First(Expr) = { (, ID, NUMBER, STRING, fst, snd }
First(Expr') = { !=, <, <=, ==, >, >=, Ɛ }
First(RelOp) = { !=, <, <=, ==, >, >= }
First(ArithExpr) = { (, ID, NUMBER, STRING, fst, snd }
First(ArithExpr') = { +, -, Ɛ }
First(Term) = { (, ID, NUMBER, STRING, fst, snd }
First(Term') = { *, /, Ɛ }
First(Factor) = { (, ID, NUMBER, STRING, fst, snd }
First(IfStatement) = { if }
First(ElsePart) = { else, Ɛ }
First(ForStatement) = { for }
```

### Follow množice

```
Follow(Program) = { $ }
Follow(CityDecl) = { $ }
Follow(CityElementList) = { } }
Follow(CityElement) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(StatementList) = { } }
Follow(Statement) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(RoadDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(BuildingDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(RadioDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(PlaylistDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(ParkDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(RiverDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(BridgeDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(RestaurantDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(CommandList) = { } }
Follow(Command) = { bend, box, circ, line, } }
Follow(PlaylistBody) = { } }
Follow(PlaylistEntry) = { add, set, } }
Follow(MetadataList) = { } }
Follow(SetCommand) = { add, set, } }
Follow(LetDecl) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(Assignment) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(Point) = { ), ,, ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, {, } }
Follow(Expr) = { ), ,, ;, to, { }
Follow(Expr') = { ), ,, ;, to, { }
Follow(RelOp) = { (, ID, NUMBER, STRING, fst, snd }
Follow(ArithExpr) = { !=, ), ,, ;, <, <=, ==, >, >=, to, { }
Follow(ArithExpr') = { !=, ), ,, ;, <, <=, ==, >, >=, to, { }
Follow(Term) = { !=, ), +, ,, -, ;, <, <=, ==, >, >=, to, { }
Follow(Term') = { !=, ), +, ,, -, ;, <, <=, ==, >, >=, to, { }
Follow(Factor) = { !=, ), *, +, ,, -, /, ;, <, <=, ==, >, >=, to, { }
Follow(IfStatement) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(ElsePart) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
Follow(ForStatement) = { ID, bridge, building, for, if, let, park, playlist, radio, restaurant, river, road, } }
```

---

## Testni primeri

### Primer 1: Osnovno mesto z cestami in zgradbami

```
city "Mestece" {
    road "Ulica" {
        line((1,0), (2,3));
    }
    
    road "Potka" {
        line((2,3), (3,4));
    }
    
    building "Zgradba" {
        box ((3, 1), (5, 4));
    }
    
    building "Hiša" {
        box((5,7), (2,3));
    }
}
```

### Primer 2: Mesto z radijsko postajo

```
city "Radio" {
    road "Glavna" {
        line ((0, 0), (10, 0));
    }
    
    radio "RadioCenter" (5, 5) {
        set ("volume", 7);
        set ("frequency", 101.2);
    }
    
    building "Zgradba" {
        box ((4, 1), (5, 4));
    }
}
```

### Primer 3: Mesto z reko in mostom

```
city "Reka" {
    river "Rečica" {
        bend ((0, 0), (5, 5), 45);
    }
    
    bridge "Most" {
        line ((1,5), (4,0));
    }
    
    restaurant "Picerija" (3, 3);
    restaurant "Slaščičarna" (4, 4);
    
    building "Zgradba" {
        box ((6, 3), (7, 4));
    }
    
    building "Zgradba2" {
        box ((8, 8), (9, 9));
    }
    
    road "Pot" {
        line ((3, 2), (5, 8));
    }
}
```

### Primer 4: Uporaba zanke

```
city "Zanka" {
    let n = 3;
    
    for i = 1 to n {
        building "Zgradba" {
            box ((i * 3, 1), (i * 3 + 2, 4));
        }
    }
    
    road "Potka" {
        line ((0, 2), (3, 2));
    }
}
```

### Primer 5: Pogojno vejanje

```
city "Pogoj" {
    let n = 3;
    
    for i = 1 to n {
        building "Zgradba" {
            box ((i * 3, 1), (i * 3 + 2, 4));
        }
        
        if i == 2 {
            park "MaliPark" {
                box ((i * 3 + 2, 1), (i * 3 + 4, 3));
            }
        }
    }
    
    restaurant "Picerija" (7, 7);
}
```

### Primer 6: Spremenljivke in zanke

```
city "mestece" {
    let count = 5;
    
    for i = 1 to count {
        building "Zgradba" {
            box ((i * 3, 1), (i * 3 + 2, 4));
        }
    }
    
    restaurant "Picerija" (7, 7);
}
```

### Primer 7: Glasba in seznami predvajanja

```
city "Glasba" {
    radio "HITFM" (3, 4) {
        set ("frequency", 102.3);
        set ("genre", "jazz");
    }
    
    playlist "VečerniMiks" (7, 9) {
        add ("pesemEna", "izvajalecEna");
        add ("pesemDva", "izvajalecDva");
        set ("naključno", 1);
    }
}
```

### Primer 8: Parki in naravni elementi

```
city "Bloki" {
    park "CentralniPark" {
        box ((0, 0), (10, 10));
    }
    
    river "ZelenaReka" {
        line ((1, 1), (25, 1));
    }
    
    bridge "KameniMost" {
        box ((12, -5), (18, 5));
    }
    
    restaurant "TestenineKraj" (10, 20);
}
```

### Primer 9: Krivulje in krogi

```
city "Primer" {
    road "JavorjevoNasipje" {
        bend ((15, 0), (15, 5), 20);
    }
    
    park "ZeleniPark" {
        circ ((25, 25), 12);
    }
}
```

### Primer 10: Kompleksnejši primer

```
city "PrimerMesta" {
    road "HrastovaCesta" {
        line ((0, 0), (40, 0));
    }
    
    park "SončniPark" {
        box ((15, 15), (35, 35));
    }
    
    restaurant "HranaKot" (22, 18);
}
```