# Pixi

A simple webapp for sharing and storing files. Think dropbox but better


# Spletna aplikacija Pixi in strežniški sistem pixiServ

Projekt pri predmetu:

Seminar iz načrtovanja in razvoja programske opreme v telekomunikacijah

_Ljubljana, 2020_

Peter Kmecl, Blaž Bertalanič


# Povzetek

Pri predmetu »Seminar iz načrtovanja in razvoja programske opreme v telekomunikacijah« sva izdelala spletno storitev za shranjevanje, organizacijo in deljenje datotek. Spletna storitev je sestavljena iz namenskega strežniškega sistema ter spletne in mobilne aplikacije.

Namesto REST storitve, sva kot osnovo za API uporabila gRPC – odprtokodno rešitev za pošiljanje binarnih ukazov v obliki ProtoBuf. Server je namenski – narejen prav za najino aplikacjo - in tako posreduje spletno dostopne vsebine, kot tudi procesira klice na API. Napisan je v jeziku golang.

Spletna aplikacija je zgrajena na osnovi knjižnic jQuery in isotope.js. Večina funkcij je napisanih prav za to aplikacijo, in se na knjižnice sklicuje le za lažjo berljivost kode. Stili so zaradi večje berljivosti napisani v jeziku SCSS.

Storitev pixiServ omogoča shranjevanje in deljenje datotek preko spletnega vmesnika Pixi. Vmesnik je zasnovan podobno kot upravljalci datotek na sistemih Windows, Linux ali Macintosh.  Tako spletna aplikacija Pixi omogoča:

- nalaganje datotek na strežnik
- prikaz slikovnih datotek
- ustvarjanje map in podmap
- sortiranje datotek po različnih atributih
- iskanje po imenu in tipu datoteke
- brisanje datotek
- označevanje priljubljenih datotek
- pregled osnovnih informacij datotek
- deljenje datotek z drugimi uporabniki preko spletne povezave
- preimenovanje datotek
- …



**Ključne besede:** deljenje datotek, upravljalec datotek, cloud storage, gRPC, protoBuf, Pixi, pixiServ

1. 1.Uvod

Pri predmetu »Seminar iz načrtovanja in razvoja programske opreme v telekomunikacijah« je bilo potrebno izdelati spletno storitev ter vsaj dva odjemalca (spletno in mobilno aplikacijo). Odločila sva se za izdelavo aplikacije, ki omogoča shranjevanje, organizacijo in deljenje datotek. V okviru spletne aplikacije sva sicer predvidela uporabo reaktivnih stilov, a sva po zahtevah seminarske naloge vseeno razvila tudi mobilno aplikacijo za mobilni operacijski sistem Android.

Za izdelavo aplikacije sva uporabila sledeče tehnologije:

- API klici: gRPC-web, gRPC, protoBuf
- Namenski strežnik: napisan v jeziku golang, tus
- Podatkovna baza: mariaDB
- Spletna aplikacija: jQuery, isotope.js, SCSS, tus.js, gRPC-web
- Mobilna aplikacija: Kotlin, tus, gRPC
- Sledenje verzij: gitLab, git

# Uporabljene tehnologije

2.1 gRPC

To je moderen odprtokodni RPC (»Remote procedure call«), ki je izjemno zmogljiv in se lahko v nespremenjeni obliki uporablja na različnih napravah in v različnih programskih jezikih. gRPC se vedno hitreje seli v podatkovne centre in spletne storitve, ki zahtevajo učinkovito komunikacijo, saj je, v primerjavi z JSON, paket gRPC do 4x manjši in hkrati prenos do 6x hitrejši od JSONa. Komunikacija gRPCja deluje preko novejšega HTTP/2 protokola, ki je varnejši, hitrejši in učinkovitejši od predhodnika HTTP/1.1.

gRPC je zasnovan na »protocol bufferjih« (protobuf), ki so Googlov mehanizem za serializacijo strukturiranih podatkov, ki so hkrati neodvisni od programskega jezika in platform. V protobufu definiramo tako storitev kot tudi sporočila, ki se uporabljajo za komunikacijo med napravami.


Kot je razvidno iz zgornje slike mi definiramo zahtevek in odgovor te to potem vnesemo v rpc klic. Torej že v Protobufu definiramo tako zahtevek kot tudi odgovor zaradi česar lahko enake klice uporabljamo v različnih programskih jezikih in sistemih, saj jih moramo le z ustreznim prevajalnikom prevesti v željeno obliko. Tako na primer se lahko med sabo pogovarjata aplikacija spisana v Javi in C++ brez kakršnih koli težav. Enako lahko počnemo z JSONom, toda tukaj je vse skupaj bistveno hitreje in učinkoviteje saj so klici binarni, medtem ko pri JSONu nazaj dobimo »string«.

## 2.2 TUS

TUS je odprtokodni protokol za nalaganje datotek na strežnik. Bazira na protokolu HTTP. Ustvarjen je bil v želji, da rešijo težavo nezanesljivega prenosa datotek in podpira praktično vse programske jezike. Glavni razlog, da sva se odločila za TUS je preprostosta implementacije in hkrati omogoča obnovljiv (»resumable«) prenos datotek na strežnik v primeru, da pride do nepredvidene prekinitve povezave med strežnikom in uporabnikom storitve. V kombinaciji z overjanjem preko gRPC je ta protokol ena glavnih komponent strežniškega sistema pixiServ.

## 2.3 Golang

Golang ali krajše Go je odprtokodni programski jezik, ki so ga pred dobrimi 10 leti razvili pri Googlu in iz dneva v dan raste v popularnosti. Gre za jezik, ki je bil zgrajen v misli na vzporednost in hkrati preprostosti za uporabo. Z že vgrajenimi sistemi za upravljanje z več nitmi (»multi thread«) omogoča razširljivost strežniškega sistema in z vgrajenimi knjižnicami predstavlja močno orodje za pisanje zalednega dela spletnih aplikacij. Zaradi preprostega upravljanje povezav omogoča hkraten dostop več uporabnikom brez večjih obremenitev na strežnik.

# gRPC (API) klici strežniškega sistema pixiServ

Storitev pixiServ uporablja nekaj manj kot 30 različnih možnih gRPC klicev za pravilno izvajanje storitve. Klici so razdeljeni na 7 podskupin:

- Registracija in overjanje uporabnika
- Prikaz datotečnega sistema in pridobivanje podatkov o njem
- Nalaganje datotek
- Upravljanje z datotekami
- Upravljanje z mapami
- Prenos datotek
- Deljenje datotek



## 3.1 Uporabljeni gRPC klici za registracija in overjanje uporabnika

### Registracija uporabnika

~~~~
message registerUserRequest{    
    string username = 1;    
    string passHash = 2;    
    string email = 3;    
    repeated string fullName = 4;
}
message loginUserResponse{    
    string authToken = 1;    
    string username = 2;
}
message getAvailableUserRequest{    
    string username = 1;
}
message getAvailableEmailRequest{    
    string email = 1;
}
rpc registerUser (registerUserRequest) returns (loginUserResponse);
rpc getAvailableUser(getAvailableUserRequest) returns(ack);
rpc getAvailableEmail(getAvailableEmailRequest) returns(ack);  
~~~~ 

Med izpolnjevanjem obrazca za registracijo spletna aplikacija pošilja poizvedbo na strežnik, če uporabniško ime ali e-mail že obstajata. V getAvailableUserRequest pošljemo uporabniško ime nazaj pa dobimo potrditev v primeru, da uporabnik še ne obstaja, v nasprotnem primeru dobimo napako, ki pomeni, da mora uporabnik uporabiti drugačno uporabniško ime. Enako se zgodi z getAvailableEmailRequest, kjer v sporočilu pošljemo vneseni e-naslov. Pri registraciji v sporočilu pošljemo uporabniško ime, zgoščeno geslo, e-naslov in polno ime uporabnika v obliki. Ob uspešni registraciji preko registerUserRequesta uporabnik nazaj prejme avtentikacijski žeton in potrjeno uporabniško ime.

### Prijava uporabnika

~~~~
message loginUserRequest{    
    string username = 1;    
    string passHash = 2;    
    string browserID = 3;    
    uint32 timestamp = 4;
}
message loginUserResponse{    
    string authToken = 1;    
    string username = 2;
}
rpc loginUser (loginUserRequest) returns (loginUserResponse);
~~~~

Pri prijavi uporabnika v sporočilu uporabniško ime, zgoščeno geslo, ID brskalnika in časovni žig. V odgovoru uporabnik prejme avtentikacijski žeton in potrjeno uporabniško ime.

### Odjava uporabnika
~~~~
message logoutUserRequest{    
    string authToken = 1;    
    string user = 2;
}
message ack{    
    bool ack = 1;
}
rpc logoutUser (logoutUserRequest) returns (ack);  
~~~~

Uporabnik pri odjavi na strežnik pošlje svoj avtentikacijski žeton in uporabniško ime. V odgovor dobi potrditev, strežnik pobriše žeton v podatkovni bazi in preusmeri uporabnika na začetno stran.

### Menjava gesla
~~~~
message changePasswordRequest{    
    string authToken = 1;    
    string user = 2;    
    string newPassHash = 3;
} 
message ack{    
    bool ack = 1;
}
rpc changePassword(changePasswordRequest) returns(ack);  
~~~~

V sporočilu za menjavo gesla pošljemo avtentikacijski žeton, uporabniško ime in novo zgoščeno geslo. V primeru potrjene overitve s strani strežnika le ta vnese novo geslo v podatkovno bazo in nazaj posreduje potrditveno sporočilo za uporabnika.



## 3.2 Prikaz datotečnega sistema in pridobivanje podatkov o njem

Vsi klici uporabljajo spodnji objekt fileSystemObject (FSO), v katerega strežnik naloži podatke o posamezni datoteki in v primeru map tudi podatke o datotekah/mapah v njej.
~~~~
message fileSystemObject{    
    string fileID = 1;    
    string fileName = 2;    
    string owner = 3;    
    bool isFolder = 4;    
    bool isFavourite = 5;    
    float fileSize = 6;    
    string fileType = 7;    
    string createdOn = 8;    
    string modifiedOn = 9;    
    repeated fileSystemObject children = 10;
} 
~~~~

FSO vsebuje unikaten ID datoteke, ime, lastnika, oznako za preverjanje ali je mapa, oznako ali je na seznamu priljubljenih, velikost in tip datoteke.

### Prenos podatkov in elementov posamezne datoteke ali mape
~~~~
message fileInfoRequest{    
    string authToken = 1;    
    string user = 2;    
    string fileID = 3;
}
message fileSystemStructureResponse{    
    string rootFolderID = 1;    
    int32 numFiles = 2;    
    string systemTime = 3;    
    repeated fileSystemObject 
    children = 4;
}
rpc getFileInfo(fileInfoRequest) returns (fileSystemStructureResponse);
~~~~

Za prenos podatkov o posamezni datoteki/mapi na strežnik uporabnik posreduje avtentikacijski žeton, svoje uporabniško ime in unikatni ID datoteke/mape katere podatke potrebuje. V odgovor dobi svojo pozicijo v strukturi, svojega nadrejenega, število datotek (v primeru mape koliko datotek je v njej, sicer le za eno datotek), čas na strežniku in FSO z ostalimi podatki o datoteki, oziroma v primeru mape tudi podatki o datotekah katerim je ta mapa nadrejena.



### Prenos celotnega datotečnega sistema
~~~~
message fileSystemStructureRequest{    
    string authToken = 1;    
    string user = 2;
}
message fileSystemStructureResponse{    
    string rootFolderID = 1;    
    int32 numFiles = 2;    
    string systemTime = 3;    
    repeated fileSystemObject children = 4;
}
rpc getFileStructureRecursive (fileSystemStructureRequest) returns(fileSystemStructureResponse);
~~~~

Uporabnik kliče ta klic takoj na začetku, ko se prijavi v aplikacijo. Nazaj prejme podatke o celotni strukturi datotek in map, ki jih ima shranjene preko FSO. Pridobi tudi podatek o številu datotek, sistemskem času in ID mape, ki je najvišje v strukturi.



### Prikaz datotek, ki so le označene kot izbrisane a jih uporabnik še lahko obnovi
~~~~
message getDeletedFilesRequest{    
    string authToken = 1;    
    string user = 2;
}
message getDeletedFilesResponse{    
    repeated fileSystemObject FSO = 1;
}
rpc getDeletedFiles(getDeletedFilesRequest) returns(getDeletedFilesResponse);
~~~~

Klic za prikaz datotek, ki so bile premaknjene v koš. Strežnik vrne FSO z vsemi datotekami, ki se tam nahajajo.

## 3.3 Nalaganje datotek

### Ustvari datoteko (naloži datoteko)
~~~~
message createFileRequest{    
    string authToken = 1;    
    string user = 2;    
    string name = 3;    
    fileType type = 4;    
    repeated string path = 5;
}
message createFileResponse{    
    string fileID = 1;    
    string uploadToken = 2;
}
rpc createFile(createFileRequest) returns(createFileResponse);
~~~~

Pred dejanskim nalaganjem datoteke na strežnik preko TUS protokola uporabnik na strežnik pošlje zahtevo za nalaganje v obliki svojega avtentikacijskega žetona, uporabniškega imena, imena datoteke, tipa datoteke in mesto (mapo) kamor želi naložiti novo datoteko. Strežnik odgovori z ustvarjenim unikatnim ID-jem, ki bo pripadal tej datoteki in žetonom potrebnim za nalaganje preko TUS-a. Uporabnik potem v »metadato« za TUS vnese ime datoteke, ki jo nalaga, tip, ID, ki ga je prejel od strežnika in žeton za nalaganje in vse skupaj pošlje na [https://pixi.tk/files/?token=](https://pixi.tk/files/?token=)»token za nalaganje«

## 3.4 Upravljanje z datotekami

### Izbriši datoteke
~~~~
message deleteFileRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string fileID = 3;    
    bool permanent = 4;
}
message ack{    
    bool ack = 1;
}
rpc deleteFile(deleteFileRequest) returns (ack);
~~~~

Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, seznam unikatnih ID-jev datotek, ki jih želi izbrisati in oznako ali želi datoteke premakniti v koš ali jih dokončno izbrisati. Strežnik odgovori s potrditvijo zahteve.



### Obnovi izbrisane datoteke
~~~~
message restoreDeletedFilesRequest{    
    string authToken = 1;    
    string user = 2;    
    string newParentID = 3;    
    repeated string fileID = 4;
}
message ack{    
    bool ack = 1;
}
rpc restoreDeletedFiles(restoreDeletedFilesRequest) returns(ack);
~~~~

Zahtevek za obnovitev datotek, ki so v košu. Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, unikatni ID mape v katero želi obnoviti datoteke in seznam ID-jev datotek, katere želi obnoviti. Strežnik odgovori s potrditvijo, ko zaključi zahtevek.

### Kopiraj datoteke
~~~~
message copyFileRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string fileID = 3;    
    repeated string newPath = 4;
}
message copyFileResponse{    
    repeated string fileID = 1;    
    repeated string newName = 2;
}
rpc copyFile(copyFileRequest) returns(copyFileResponse);
~~~~

Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, seznam ID-jev datotek, ki jih želi skopirati in ID-mape v katero želi skopirati te datoteke. Strežnik odgovori s seznamom ID-jev novih prekopiranih datotek in njihovimi imeni. V primeru podvojenega imena doda ustrezno oznako.

### Premakni datoteke
~~~~
message moveFileRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string fileID = 3;    
    repeated string newPath = 4;
}
message ack{    
    bool ack = 1;}  
rpc moveFile(moveFileRequest) returns (ack);
~~~~

Uporabnik lahko datoteke premika iz ene mape v drugo. Na strežnik pošlje zahtevek s svojim avtorizacijskim žetonom, uporabniškim imenom, seznamom ID-jev datotek, ki jih želi prestaviti in ID nove mape kamor želi prestaviti datoteke. Strežnik po opravljenem premiku pošlje nazaj potrditev.



### Dodaj datoteko pod priljubljene
~~~~
message setFileFavouriteRequest{    
    string authToken = 1;   
    string user = 2;    
    bool isFavourite = 3;    
    repeated string fileID = 4;
}
message ack{    
    bool ack = 1;
}
rpc setFileFavourite(setFileFavouriteRequest) returns(ack);
~~~~

Uporabnik na strežnik pošlje zahtevek za spremembo statusa »priljubljeno« v datoteki. V sporočilu pošlje svoj avtentikacijski žeton, uporabniško ime, oznako ali naj datoteko uvrsti na seznam priljubljenih ali jo od tam odstrani in seznam ID-jev datotek, katerim želi spremeniti »priljubljeni« status. Strežnik odgovori s potrditvijo, ko naredi spremembo.

### Preimenuj datoteko
~~~~
message renameFileRequest{    
    string authToken = 1;    
    string user = 2;    
    string fileID = 3;    
    string newName = 4;
}
message ack{    
    bool ack = 1;
}
rpc renameFile(renameFileRequest) returns(ack);
~~~~

Uporabnik, za preimenovanje datoteke, na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, ID datoteke, ki ji želi spremeniti ime in novo ime. Strežnik po opravljeni menjavi pošlje potrditveno sporočilo.

## 3.5 Upravljanje z mapami

### Ustvari mapo
~~~~
message createFolderRequest{    
    string authToken = 1;    
    string user = 2;    
    string name = 3;    
    repeated string path = 4;
}
message createFolderResponse{    
    string folderID = 1;
}
rpc createFolder(createFolderRequest) returns(createFolderResponse);
~~~~

Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, ime nove datoteke in ID mape v kateri želi ustvariti datoteko. Strežnik odgovori z ustvarjenim ID-jem nove mape.



### Izbriši mapo
~~~~
message deleteFolderRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string folderID = 3;
}
message ack{    
    bool ack = 1;
}
rpc deleteFolder(deleteFolderRequest) returns (ack);  |
~~~~

Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, seznam unikatnih ID-jev map, ki jih želi izbrisati. Strežnik po opravljeni storitvi odgovori s potrditvijo zahteve.

### Kopiraj mapo
~~~~
message copyFolderRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string folderID = 3;    
    repeated string newPath = 4;
} 
message copyFolderResponse{    
    repeated string folderID = 1;    
    repeated string newName = 2;
}
rpc copyFolder(copyFolderRequest) returns(copyFolderResponse);
~~~~

Uporabnik na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, seznam ID-jev map, ki jih želi skopirati in ID-mape v katero želi skopirati te mape in njeno vsebino. Strežnik odgovori s seznamom ID-jev novih prekopiranih map in vsebin v njih in njihovimi imeni. V primeru podvojenega imena doda ustrezno oznako.

### Premakni mapo
~~~~
message moveFolderRequest{    
    string authToken = 1;    
    string user = 2;    
    repeated string folderID = 3;    
    repeated string newPath = 4;
}
message ack{    
    bool ack = 1;
}
rpc moveFolder(moveFolderRequest) returns (ack);
~~~~

Uporabnik lahko mape premika iz ene mape v drugo. Na strežnik pošlje zahtevek s svojim avtorizacijskim žetonom, uporabniškim imenom, seznamom ID-jev map, ki jih želi prestaviti in ID nove mape kamor jih želi prestaviti. Strežnik po opravljenem premiku pošlje nazaj potrditev.



### Združi mape
~~~~
message mergeFolderRequest{    
    string authToken = 1;    
    string user = 2;    
    string masterFolderID = 3;    
    string slaveFolderID = 4;
}
message ack{    
    bool ack = 1;
}
rpc mergeFolder(mergeFolderRequest) returns(ack);
~~~~

Uporabnik lahko združi vsebino dveh map tako, da na strežnik pošlje zahtevek s svojim avtorizacijskim žetonom, uporabniškim imenom, ID-jem mape, ki bo ostala po združitvi in ID-jem mape katere vsebina se bo tja prestavila.

### Dodaj mapo pod priljubljene
~~~~
message setFileFavouriteRequest{    
    string authToken = 1;    
    string user = 2;    
    bool isFavourite = 3;    
    repeated string fileID = 4;
}
message ack{    
    bool ack = 1;
}
rpc setFileFavourite(setFileFavouriteRequest) returns(ack);
~~~~



Uporabnik na strežnik pošlje zahtevek za spremembo statusa »priljubljeno« v mapi. V sporočilu pošlje svoj avtentikacijski žeton, uporabniško ime, oznako ali naj datoteko uvrsti na seznam priljubljenih ali jo od tam odstrani in seznam ID-jev map, katerim želi spremeniti »priljubljeni« status. Strežnik odgovori s potrditvijo, ko naredi spremembo.

### Preimenuj mapo
~~~~
message renameFileRequest{    
    string authToken = 1;    
    string user = 2;    
    string fileID = 3;    
    string newName = 4;
}
message ack{    
    bool ack = 1;
}
rpc renameFile(renameFileRequest) returns(ack);
~~~~

Uporabnik, za preimenovanje mape, na strežnik pošlje svoj avtentikacijski žeton, uporabniško ime, ID mape, ki ji želi spremeniti ime in novo ime. Strežnik po opravljeni menjavi pošlje potrditveno sporočilo.



## 3.6 Prenos datotek

### Zahtevek za prenos
~~~~
message getAccessTokenRequest{    
    string authToken = 1;    
    string user = 2;    
    string fileID = 3;
}
message getAccessTokenResponse{    
    string fileID = 1;    
    string accessToken = 2;
}
rpc getAccessToken(getAccessTokenRequest) returns (getAccessTokenResponse);
~~~~

Ko uporabnik želi prenesti datoteko, zaprosi za prenos in v zahtevku pošlje svoj avtentikacijski žeton, uporabniško ime, ID datoteke, ki jo želi prenesti. Strežnik posreduje nazaj ID datoteke, da potrdi da  gre za enako ter žeton za prenos, ki ga uporabnik nato posreduje na [https://pixi.tk/download/?token=](https://pixi.tk/download/?token=)»token za prenos«



## 3.7 Deljenje datotek

### Zahtevek za ustvarjanje povezave za deljenje
~~~~
message getShareTokenRequest{    
    string authToken = 1;   
    string user = 2;    
    string fileID = 3;
} 
message getShareTokenResponse{    
    string shareToken = 1;
}
rpc getShareToken(getShareTokenRequest) returns(getShareTokenResponse);
~~~~

Uporabnik lahko svoje datoteke deli naokoli, tako da pošlje zahtevek za delitveno povezavo. Na strežnik pošlje zahtevek s svojim avtentikacijskim žetonom, uporabniškim imenom in ID-jem datoteke, ki jo želi deliti z drugimi. Strežnik vrne žeton, ki ga uporabnik potem pripne povezavi [https://pixi.tk/s/](https://pixi.tk/s/)»pridobljen žeton«



# 4. Spletna aplikacija Pixi

## 4.1 Struktura spletne aplikacije

Spletna aplikacija Pixi je sestavljena iz zalednega dela spisanega v Golangu, medtem ko je uporabniški vmesnik narejen v SCSS, Javascript, HTML, jQuery, … in je povsem prilagodljiv neglede na tip uporabljene naprave ter njene velikosti zaslona. Zaledni del skrbi tudi za streženje spletnih vsebin poleg že znane funkcije upravljanja z API klici. Aplikacija uporablja tudi Reverse Proxy, ki ga poganja Apache. Celoten sitem je samozadosten in posledično preprost za namestitev na poljubnem strežniku s poljubnimi tehnologijami.


## 4.2 Strani spletne aplikacije Pixi

### 4.2.1 Prijavna stran


Začetna stran, na kateri se odvrti animacija, ki poskrbi za boljšo uporabniško izkušnjo medtem kot brskalnik naloži vse potrebne skripte za nemoteno delovanje spletne aplikacije.

Prijavni zaslon ima animirano ozadje, ki se naključno premika po ekranu. Okno za prijavo omogoča tudi izbor obrazca za registracijo.

###
4.2.2 Glavni zaslon aplikacije

Po prijavi se nam odpre datotečni brskalnik, ki je osnovne koncepte črpal iz že znanih brskalnikov, ki so implementirani v vseh popularnih operacijskih sistemih. Na levi strani imamo drevesno postavitev celotnega sistema in lahko hitreje preskakujemo med ustvarjenimi mapami. Prav tako imamo možnost skočiti v mapi Koš in Priljubljeno, kjer se nam prikažejo le datoteke, ki so bile izbrisane oziroma označene kot priljubljene. Na vrhu se nam v orodni vrstici, glede na trenutno pozicijo v sistemu, vedno izpisuje celotna pot, kje se trenutna mapa nahaja. Na desni strani, orodne vrstice imamo gumbe, ki nam omogočajo izbris in prenos označenih datotek, nalaganje novih vsebin, dodajanje izbranih datotek pod priljubljene, ustvarjanje novih map, sortiranje vsebine po velikosti, imenu, tipu, …, in na koncu možnost prikaza osnovnih informacij označenih objektov. Vse te funkcionalnosti so prisotne tudi v meniju, ki se prikaže če na datoteke pritisnemo z desnim klikom miške. V desnem zgornjem kotu je prisoten spustni meni, kjer lahko uporabnik zamenja geslo ali se odjavi iz aplikacije.

# 5. Mobilna aplikacija Pixi

Mobilna aplikacija Pixi je spisana v Kotlinu in vsebuje podobne funkcionalnosti kot matična spletna aplikacija. Za prikaz uporablja RecyclerView, medtem ko je vizualna podoba zasnovana tako, da omogoča podobno interakcijo kot nekatere druge aplikacije za upravljanje z datotekami in mapami na mobilnem telefonu.

## 5.1 Začetni zaslon aplikacije

Začetni zaslon aplikacije je zasnovan podobno kot na matični strani kjer je za ozadje sprogramiran svoj objekt, ki omogoča animirano ozadje kot na spletu. Manjka registracijski obrazec, kar je namerna odločitev, saj je želja, da se uporabniki prvič prijavijo preko spletne aplikacije.

## 5.2 Uporabniški zaslon


Na uporabniškem zaslonu se nam takoj ob prijavi naložijo vse datoteke in mape, ki jih imamo spravljene v matični mapi na strežniku.

Na desni strani imamo možnost izbire na kakšen način želimo prikazati vrstni red vsebine (po velikosti, tipu, imenu, …) in gumb za iskanje po prikazani vsebini trenutne mape. Spodaj lahko med tremi gumbi izbiramo ali želimo odpreti mapo s priljubljenimi, primarni prikaz vseh datotek ali mapo s pobrisanimi datotekami. V levem zgornjem kotu se nam odpre meni v katerem lahko izbiramo med nalaganjem datotek, ustvarjanjem nove mape, enako lahko izbiramo med mapo priljubljeni, koš in klasičen pogled, ki je povezan na spodnjo orodno vrstico in tako je vedno označen pogled, ki je trenutno prikazan. Na koncu je še gumb za odjavo, ki uporabnika preusmeri na začetni zaslon.


Pri vsaki datoteki/mapi je gumb (tri pokončne pikice), ki odpre meni za upravljanje s posamezno datoteko/mapo. Tam je mogoče izbrati predogled slik, prenesti datoteko, preimenovati datoteko/mapo, dodati vsebino pod priljubljene, ustvariti povezavo za deljenje, izbrisati datoteko oziroma jo prestaviti v koš in prikazati osnovne informacije izbranega elementa.



# 6. Zaključek

Celotna storitev pixiServ je bila zasnova na čim boljši razširljivosti in predvsem učinkovitosti za uporabnika. Sistem temelji na odprtokodnih rešitvah kot so TUS, Protocol Buffer in gRPC. Zaradi uporabe programskih jezikov kot sta Golang in javascript je celoten sitem samozadosten in posledično preprost za namestitev na poljubnem strežniku s poljubnimi tehnologijami. Za celotno nalogo sva se odločila predvsem zato, da sva v praksi pozkusila ustvariti svoj brskalnik datotek in na tak način osvojiti koncepte delovanja takšnih sistemov, ki so prisotni v našem vsakodnevnem življenju. Storitev predvideva manipulacijo z datotekami in mapami, omogoča razvrščanje vsebine, nalaganje novih in prenos že naloženih datotek kot tudi deljenje z drugimi ljudmi preko posebne povezave. Obe aplikaciji, ki uporabljata storitev pixiServ, sta dizajnirani po smernicah modernih datotečnih brskalnikov in implementirata veliko podobnih funkcionalnosti.