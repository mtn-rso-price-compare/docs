# Primerjalnik cen izdelkov
**Avtor**: Mihael Rajh (skupina 72)

**Opis projekta**: Namen izdelane aplikacije je primerjava cen izdelkov med različnimi trgovinami. Uporabnikom med drugim omogoča izbiro priljubljenih izdelkov in sestavo poljubnih košaric oziroma naborov izdelkov. Cene se lahko primerjajo bodisi za posamezen izdelek, ali pa za košarice v celoti.

**Razvojno okolje**: Pri izdelavi projekta je bilo uporabljeno Javansko ogrodje KumuluzEE, Apache Maven in Docker tehnologija. Pri razvoju so bili uporabljeni IDE IntelliJ IDEA Ultimate, Docker Desktop in Postman. Za namestitev so bili uporabljeni GitHub Actions, Docker Hub in Microsoft Azure.



## Arhitektura mikrostoritev

![Arhitekturni diagram](figures/Arhitekturni_diagram.png)



| Mikrostoritev        | Ažuriranje in hranjenje cen                                  |
| -------------------- | ------------------------------------------------------------ |
| Opis funkcionalnosti | Primarni cilj mikrostoritve je hranjenje cen izdelkov v podatkovni bazi in strežba podatkov aplikacijam, ki te informacije potrebujejo. Sekundarni cilj mikrostoritve je iskanje in posodabljanje cen glede na ažurno stanje izbranih trgovinskih spletnih strani. |
| GitHub               | [mtn-rso-price-compare/price-updater](https://github.com/mtn-rso-price-compare/price-updater) |
| Docker Hub           | [mikethenut/price-updater](https://hub.docker.com/repository/docker/mikethenut/price-updater) |
| API vmesnik          | */item*: Podatki o vseh izdelkih v bazi (**GET**).<br>*/item/{itemId}*: Podatki o cenah artikla {itemId} (**GET**).<br>*/store*: Podatki o vseh trgovinah v bazi (**GET**).<br>*/store/{storeId}*: Podatki o trgovini {storeId} in njihovih izdelkih (**GET**).<br>*/request*: Ukazi za takojšnje ažuriranje cen (**GET, POST**).<br>*/request/{requestId}*: Podatki o izvedbi ukaza za ažuriranje cen (**GET**). |
| OpenAPI              | /openapi                                                     |
| Swagger UI           | /openapi/ui?url=<base-url>/openapi#/                         |
| Metrike              | /metrics                                                     |
| Kontrole zdravja     | /health                                                      |



| Mikrostoritev        | Upravljanje naborov                                          |
| -------------------- | ------------------------------------------------------------ |
| Opis funkcionalnosti | Mikrostoritev je namenjena hranjenju podatkov o naborih izdelkov uporabnika, imenovani košarice. Hrani tudi podake o kategorijah izdelkov, ki predstavljajo ločeno vrsto naborov. Mikrostoritev omogoča poizvedbe za celotne nabore glede na cene, ki jih vrača Ažuriranje in hranjenje cen. |
| GitHub               | [mtn-rso-price-compare/collection-manager](https://github.com/mtn-rso-price-compare/collection-manager) |
| Docker Hub           | [mikethenut/collection-manager](https://hub.docker.com/repository/docker/mikethenut/collection-manager) |
| API vmesnik          | */collection*: Podatki o vseh nakupovalnih seznamih (**GET, POST**).<br>*/collection/{collectionId}*: Podatki o nakupovalnem seznamu {collectionId} (**GET, POST, PUT, DELETE**).<br>*/collection/{collectionId}/{itemId}*: Izdelek {itemId} na nakupovalnem seznamu {collectionId} (**DELETE**).<br>*/tag*: Seznam vseh oznak izdelkov v bazi (**GET, POST**).<br>*/tag/{tagId}*: Podatki o izdelkih z oznako {tagId} (**GET, POST, PUT, DELETE**).<br>*/tag/{tagId}/{itemId}*: Izdelek {itemId} z oznako {tagId} (**DELETE**). |
| OpenAPI              | /openapi                                                     |
| Swager UI            | /openapi/ui?url=<base-url>/openapi#/                         |
| Metrike              | /metrics                                                     |
| Kontrole zdravja     | /health                                                      |



## Viri konfiguracije

Aplikacija se zanaša na tri različne vire konfiguracije: konfiguracijska datoteka, okoljske spremenljivke in agent Consul. V konfiguracijski datoteki so poleg splošnih podatkov navedene predvsem nastavitve implementacijske narave: nastavitve OpenAPI dokumentacije, konfiguracija beleženja dnevnikov, nekatere aktivne kontrole zdravja in zbrane metrike. Konfiguracijska datoteka je nastavljena za lokalni zagon aplikacije ob sprotnemu razvoju.

V datotekah za namestitev aplikacije so nekatere vrednosti prepisane z ustreznimi vrednostmi za namestitveno okolje. To so naslov, uporabniško ime in geslo podatkovne baze, naslov strežnika za OpenAPI specifikacijo in agenta Consul. Pri tem sta geslo podatkovne baze in naslov agenta Consul shranjena v Kubernetes objektih tipa `Secret`.

Dodatne konfiguracijske vrednosti aplikacije, ki se lahko spreminjajo med izvajanjem, so navedene v agentu Consul. Njihove privzete vrednosti se nahajajo tudi v konfiguracijskih datotekah. Vse konfiguracijske nastavitve agenta Consul za mikrostoritev Ažuriranje in hranjenje cen se nahajajo v imeniku `environments/dev/services/price-updater-service/1.0.0/config`, ki bo od tu naprej okrajšan na `$c1`, nastavitve agenta Consul za mikrostoritev Upravljanje naborov pa se nahajajo v imeniku `environments/dev/services/collection-manager-service/1.0.0/config`, ki bo od tu naprej okrajšan na `$c2`. Tu so navedene vse konfiguracijske možnosti agenta Consul in njihove privzete vrednosti.

| Mikrostoritev               | Konfiguracijska možnost                                      | Privzeta vrednost | Opis                                                         |
| --------------------------- | ------------------------------------------------------------ | ----------------- | ------------------------------------------------------------ |
| Ažuriranje in hranjenje cen | `$c1/api-properties/return-all-item-prices`                  | `true`            | Določa, ali naj mikrostoritev privzeto vrne seznam cen za posamezne izdelke tudi za zahteve po več izdelkih. Odjemalec lahko obnašanje določi eksplicitno s parametrom `returnPrice`. |
| Ažuriranje in hranjenje cen | `$c1/api-properties/return-all-store-prices`                 | `true`            | Določa, ali naj mikrostoritev privzeto vrne seznam cen za posamezne trgovine tudi za zahteve po več trgovinah. Odjemalec lahko obnašanje določi eksplicitno s parametrom `returnPrice`. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/request-retention-period`  | `604800`          | Število sekund od zadnje posodobitve, po katerem naj se zahteve za posodobitev baze cen trajno izbrišejo iz podatkovne baze. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/request-deletion-check-interval` | `3600`            | Dolžina intervala v sekundah, na katerem naj se preveri starost hranjenih zahtev za posodobitev baze cen. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/price-retention-period`    | `604800`          | Število sekund od zadnje posodobitve, po katerem naj se cene izdelkov trajno izbrišejo iz podatkovne baze. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/data-source`               | `1`               | Podatkovni vir, ki je uporabljen za posodabljanje cen v podatkovni bazi. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/update-prices-tus`         | `true`            | Določa, ali naj se ob zahtevi za posodobitev baze cen posodobijo cene trgovine Tuš. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/update-prices-spar`        | `true`            | Določa, ali naj se ob zahtevi za posodobitev baze cen posodobijo cene trgovine Spar. |
| Ažuriranje in hranjenje cen | `$c1/update-processing-properties/update-prices-mercator`    | `true`            | Določa, ali naj se ob zahtevi za posodobitev baze cen posodobijo cene trgovine Mercator. |
| Ažuriranje in hranjenje cen | `$c1/global-properties/app-liveness`                         | `true`            | Lažna nastavitev za kontrolo zdravja tipa `liveness`, ki jo preverja Kubernetes. |
| Ažuriranje in hranjenje cen | `$c1/global-properties/app-readiness`                        | `true`            | Lažna nastavitev za kontrolo zdravja tipa `readiness`, ki jo preverja Kubernetes. |
| Upravljanje naborov         | `$c2/api-properties/verify-item-exists`                      | `true`            | Določa, ali naj mikrostoritev preveri obstoj izdelka v mikrostoritvi `Ažuriranje in hranjenje cen` ob zahtevi za dodajanje izdelka na nabor ali oznako. Če izdelek ne obstaja, na nabor oz. oznako ne bo dodan. |
| Upravljanje naborov         | `$c2/api-properties/return-collection-item-information`      | `true`            | Določa, ali naj mikrostoritev privzeto vrne seznam cen za posamezne izdelke tudi za zahteve po celotnem naboru izdelkov. Odjemalec lahko obnašanje določi eksplicitno s parametrom `returnPrice`. |
| Upravljanje naborov         | `$c2/client-properties/price-updater-host`                   | `localhost:8080`  | Naslov, na katerem je dostopna mikrostoritev `Ažuriranje in hranjenje cen`. |
| Upravljanje naborov         | `$c2/global-properties/app-liveness`                         | `true`            | Lažna nastavitev za kontrolo zdravja tipa `liveness`, ki jo preverja Kubernetes. |
| Upravljanje naborov         | `$c2/global-properties/app-readiness`                        | `true`            | Lažna nastavitev za kontrolo zdravja tipa `readiness`, ki jo preverja Kubernetes. |



## Kontrole zdravja in metrike

Obe mikrostoritvi izvajata več kontrol zdravja. Mikrostoritev Upravljanje naborov izpostavlja kontrolo `ReadinessConfigHealthCheck` tipa readiness, ter kontroli `DataSourceHealthCheck` in `LivenessConfigHealthCheck` tipa liveness. Pri tem sta `ReadinessConfigHealthCheck` in `LivenessConfigHealthCheck` izpostavljeni nastavitvi `global-properties/app-readiness` ter `global-properties/app-liveness` iz agenta Consul. `DataSourceHealthCheck` preverja veljavnost povezave na podatkovno bazo.

Mikrostoritev Ažuriranje in hranjenje cen izpostavlja kontroli `RequestProcessingHealthCheck` in `ReadinessConfigHealthCheck` tipa readiness, ter kontroli `DataSourceHealthCheck` in `LivenessConfigHealthCheck` tipa liveness. Pri tem sta `ReadinessConfigHealthCheck` in `LivenessConfigHealthCheck` izpostavljeni nastavitvi `global-properties/app-readiness` ter `global-properties/app-liveness` iz agenta Consul. `DataSourceHealthCheck` preverja veljavnost povezave na podatkovno bazo, `RequestProcessingHealthCheck` pa preverja, ali se trenutno izvaja posodabljanje cen v podatkovni bazi, in v tem primeru vrne negativen odgovor.

Obe mikrostoritvi zbirata metrike na končnih točkah API-ja preko `kumuluzee.metrics.web-instrumentation` nastavitev, ki spremlja število odgovorov glede na njihov status za posamezne končne točke. Obe mikrostoritvi prav tako zbirata število klicev za vse funkcije, ki opravljajo neposredno s podatkovno bazo. Mikrostoritev Ažuriranje in hranjenje cen spremlja še čas, ki ga potrebuje posamezna posodobitev podatkovne baze. Mikrostoritev Upravljanje naborov pa spremlja še število izvedenih zahtev na mikrostoritev Ažuriranje in hranjenje cen, ter privzete metrike o tam uporabljenih mehanizmih odpora na napake.



## OpenAPI dokumentacija in centralizirano beleženje dnevnikov

Z OpenAPI anotacijami so dokumentirani so vsi viri API-jev obeh mikrostoritev. Uporabljene so bile anotacije tipa:

- ApiResponse
- ApiResponses
- Contact
- Content
- ExternalDocumentation
- Header
- Info
- OpenApiDefinition
- Operation
- Parameter
- RequestBody
- Schema
- SchemaType
- Server

Dnevniki instanc aplikacije so poslane na logit.io strežnik z ogrodjem log4j, in sicer preko appender-ja tipa UDP. Izsek konfiguracijske datoteke za beleženje dnevnikov:

```xml
<Socket name="logstash" host="ba28be7a-3aac-4a7d-8f38-685d9044b466-ls.logit.io" port="19730" protocol="udp">
    <JSONLayout complete="false" compact="true" eventEol="true" charset="UTF-8" properties="true"/>
</Socket>
```



## Namestitev na Kubernetes

Vse datoteke, ki so bile uporabljene ob namestitvi so na voljo v repozitoriju za dokumentacijo pod imenikom `deployment`. Ob času zagovora aplikacije je bila namestitev izvedena pri ponudniku Microsoft Azure. Pri tem je bila mikrostoritev Ažuriranje in hranjenje cen dostopna na naslovu `http://51.104.180.67/price-updater`, mikrostoritev Upravljanje naborov pa na naslovu `http://51.104.180.67/collection-manager`. Sledi seznam vseh nameščenih storitev v gruči:

- Privzete storitve ponudnika Kubernetes tipa `ClusterIP`: `kubernetes`, `kube-dns` in `metrics-server`.
- Storitev tipa `ClusterIP`, ki izpostavlja namestitev mikrostoritve Ažuriranje in hranjenje cen.
- Storitev tipa `ClusterIP`, ki izpostavlja namestitev mikrostoritve Upravljanje naborov.
- Storitev tipa `LoadBalancer`, ki izpostavlja namestitev agenta Consul.
- Storitev tipa `ClusterIP`, ki izpostavlja namestitev instance Prometheus.
- Storitvi tipa `ClusterIP` in `LoadBalancer`, ki izpostavljata NGINX controller za Ingress.
- Ingress ponudnika NGINX, ki preusmerja promet na izpostavljeni mikrostoritvi in instanco Prometheus.



## Primeri uporabe

- Uporabniki lahko primerjajo cene izdelkov med trgovinami.
- Uporabniki lahko ustvarijo poljubne nabore izdelkov, ki se imenujejo nakupovalni vozički.
- Uporabniki lahko primerjajo skupno ceno svojih nakupovalnih vozičkov med trgovinami.
- Uporabniki lahko označijo izdelke kot priljubljene.
- Uporabniki lahko pregledajo svoje priljubljene izdelke in jih po želji odstranijo s seznama priljubljenih.
- Uporabniki lahko primerjajo skupno ceno svojih priljubljenih izdelkov med trgovinami.

### Primer napredne uporabe

Uporabnik želi dodati predmet iz določene kategorije na svoj nakupovalni seznam. Najprej zahteva pregled izdelkov iz določene kategorije, ki jih vrne mikrostoritev `Upravljanje naborov`. Nato enega izmed izdelkov doda na svoj seznam, pri čemer najprej `Upravljanje naborov` shrani spremembo, nato pa od mikrostoritve `Ažuriranje in hranjenje cen` zahteva cene vseh izdelkov na seznamu. S tem potrdi obstoj dodanega izdelka, hkrati pa lahko uporabniku vrne podatke o cenah za celoten seznam.

![Sequence diagram](figures/Sequence_diagram.png)

### Primer mehanizma odpora na napake

V aplikaciji je povezava iz mikrostoritve Upravljanje naborov na Ažuriranje in hranjenje cen implementirana z asinhronim REST klicem. Na tem REST klicu je dodan mehanizem odpora na napake, in sicer s Fallback metodo, ki se izvede v primeru napake, in prekinjevalcem toka. Nastavljen je tako, da ima drseče okno velikosti treh zahtevkov, v odprtem stanju med poskusi čaka 60 sekund (za namene demonstracije), ob uspešno izvedenem zahtevku pa se takoj vrne nazaj v zaprto stanje.

Njegovo funkcionalnost lahko demonstriramo tako, da spreminjamo naslov mikrostoritve Ažuriranje in hranjenje cen v agentu Consul. Če naslov napačno nastavimo, bo naslednja poizvedba neuspešna. Ko to parkrat ponovimo, bo prekinjevalec toka prešel v odprto stanje. Če naslov nato hitro popravimo, bo do 60 sekund mikrostoritev še vedno vračala rezultate brez podatkov o cenah. Stanje prekinjevalca toka in število preusmerjenih zahtevkov lahko med tem spremljamo z metrikami na instanci Prometheus.



## Diagram ER

![ER diagram](figures/ER_diagram.png)



## Primeri objektov JSON 

#### /item

```json
{
    item_id:36,
    item_name:"piščančje prsi",
    price: [
    	{
    	    store_id:2,
    	    amount:7.3,
    	    last_updated:"2022-11-13T12.05"
    	},
    	{
    	    store_id:3,
    	    amount:7.6,
     	    last_updated:"2022-11-13T12.05"
    	}
    ]
}
```

Atribut 'price' je privzeto vrnjen le za zahtevke na določen item ID.

#### /store

```json
{
    store_id:1,
    store_name:"ENGROTUŠ d.o.o.",
    url:"https://www.tus.si"
    price: [
    	{
    	    item_id:36,
    	    amount:7.3,
    	    last_updated:"2022-11-13T12.05"
    	},
    	{
    	    item_id:27,
    	    amount:5.9,
     	    last_updated:"2022-11-13T12.05"
    	}
    ]
}
```

Atribut 'price' je privzeto vrnjen le za zahtevke na določen store ID.

#### /request

```json
{
    request_id:7,
    status:"SUCCESS",
    last_updated:"2022-11-13T12.05"
}
```

#### /collection

```json
{
    collection_id:9,
    user_id:2,
    collection_name:"Shopping list",
    item: [
        {
    	    item_id:36,
    	    item_name:"piščančje prsi",
         	amount:2,
        	price: [
            	{
                	store_id:2,
                	amount:7.3,
                	last_updated:"2022-11-13T12.05"
            	},
            	{
               		store_id:3,
                	amount:7.6,
                	last_updated:"2022-11-13T12.05"
            	}
            ]
        },
        {
            item_id:27,
            item_name:"sir Parmigiano Reggiano",
            amount:1,
            price: [
            	{
                	store_id:2,
                	amount:5.6,
                	last_updated:"2022-11-13T12.05"
            	},
            	{
               		store_id:3,
                	amount:5.2,
                	last_updated:"2022-11-13T12.05"
            	}
            ]
        }
    ],
    price_total: [
        {
            store_id:2,
            amount:20.2
        },
        {
            store_id:3,
            amount:20.4
        }
    ]
}
```

#### /tag

```json
{
    id:3,
    tag_name:"Meso in mesni izdelki",
    item: [
        {
    	    id:36
    	    item_name:"piščančje prsi"
        },
        {
            id:25,
            item_name:"svinjska rebra"
        }
    ]
}
```

