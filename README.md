# Primerjalnik cen izdelkov
**Avtor**: Mihael Rajh (skupina 72)

**Opis projekta**: Namen izdelane aplikacije je primerjava cen izdelkov med različnimi trgovinami. Uporabnikom med drugim omogoča izbiro priljubljenih izdelkov in sestavo poljubnih košaric oziroma naborov izdelkov. Cene se lahko primerjajo bodisi za posamezen izdelek, ali pa košaric v celoti.

**Razvojno okolje**: Pri izdelavi projekta je bilo uporabljeno ogrodje KumuluzEE v povezavi z JDK, Apache Maven in Docker tehnologijo. Za razvoj so bili uporabljeni IDE IntelliJ IDEA Ultimate, GitHub Actions in Postman.



## Arhitektura mikrostoritev

![Arhitekturni diagram](.\figures\Arhitekturni diagram.png)

### Opis mikrostoritev

| Mikrostoritev        | Ažuriranje in hranjenje cen                                  |
| -------------------- | ------------------------------------------------------------ |
| Opis funkcionalnosti | Primarni cilj mikrostoritve je hranjenje cen izdelkov v podatkovni bazi in strežba podatkov aplikacijam, ki te informacije potrebujejo. Sekundarni cilj mikrostoritve je iskanje in posodabljanje cen glede na ažurno stanje izbranih trgovinskih spletnih strani. |
| Odvisnosti           | /                                                            |
| Metrike              |                                                              |
| API vmesnik          | */item*: Podatki o cenah vseh izdelkov v bazi (**GET**).<br>*/item/{itemId}*: Podatki o cenah artikla {itemId} (**GET**).<br>*/store*: Podatki o vseh trgovinah v bazi (**GET**).<br>*/store/{storeId}*: Podatki o trgovini {storeId} in njihovih izdelkih (**GET**).<br>*/request*: Ukazi za takojšnje ažuriranje cen (**GET, POST**).<br>*/request/{requestId}*: Podatki o izvedbi ukaza za ažuriranje cen (**GET, DELETE**). |



| Mikrostoritev        | Upravljanje naborov                                          |
| -------------------- | ------------------------------------------------------------ |
| Opis funkcionalnosti | Mikrostoritev je namenjena hranjenju podatkov o naborih izdelkov uporabnika, imenovani košarice. Hrani tudi podake o kategorijah izdelkov, ki predstavljajo ločeno vrsto naborov. Mikrostoritev omogoča poizvedbe za celotne nabore glede na cene, ki jih vrača Ažuriranje in hranjenje cen. |
| Odvisnosti           | Ažuriranje in hranjenje cen (mikrostoritev)                  |
| Metrike              |                                                              |
| API vmesnik          | */collection*: Podatki o vseh nakupovalnih seznamih (**GET, POST**).<br>*/collection/{collectionId}*: Podatki o nakupovalnem seznamu {collectionId} (**GET, PUT, DELETE**).<br>*/tag*: Seznam vseh oznak izdelkov v bazi (**GET, POST**).<br/>*/tag/{tagId}*: Podatki o izdelkih z oznako {tagId} (**GET, PUT, DELETE**). |



## Primeri uporabe

- Uporabniki lahko primerjajo cene izdelkov med trgovinami.
- Uporabniki lahko ustvarijo poljubne nabore izdelkov, ki se imenujejo nakupovalni vozički.
- Uporabniki lahko primerjajo skupno ceno svojih nakupovalnih vozičkov med trgovinami.
- Uporabniki lahko označijo izdelke kot priljubljene.
- Uporabniki lahko pregledajo svoje priljubljene izdelke in jih po želji odstranijo s seznama priljubljenih.
- Uporabniki lahko primerjajo skupno ceno svojih priljubljenih izdelkov med trgovinami.

### Primer napredne uporabe

![Sequence diagram](.\figures\Sequence diagram.png)



## Diagram ER

![ER diagram](.\figures\ER diagram.png)



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

#### /store

```json
{
    store_id:1,
    store_name:"ENGROTUŠ d.o.o.",
    link:"https://www.tus.si"
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

#### /request

```json
{
	request_id:7,
	status:"COMPLETED",
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
    		item_id:36
    		item_name:"piščančje prsi"
        },
        {
            item_id:27,
            item_name:"sir Parmigiano Reggiano"
        }
    ]
}
```

#### /tag

```json
{
    id:3,
    name:"Meso in mesni izdelki",
    item: [
        {
    		id:36
    		name:"piščančje prsi"
        },
        {
            id:25,
            name:"svinjska rebra"
        }
    ]
}
```

