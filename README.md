# Load balancing

I dag skal i sætte en load-balancer op foran jeres SolR servere, og teste hvordan den performer, i sammenligning med en enkelt solr instans.

## Solr og nginx
Start med at start jeres solr cluster fra sidste gang, og sikr jer, at alle serverne kører.


Derefter skal i starte en ny server og installere nginx på den

```
sudo yum install nginx
```

...virker ikke, men kig på outputtet i terminalen, så burde i kunne regne den ud.

ngnix bliver installere som en service - Det vil sige at automatisk starter sig selv når serveren bliver startet.

Dens konfigurations-filer ligger i `/etc/nginx`

Følg derefter guiden https://www.upcloud.com/support/how-to-set-up-load-balancing/ for at sætte nginx op - Start fra afsnittet "Configuring nginx as a load balancer".

Efter du har sat nginx op skulle du gerne kunne tilgå solr via nginx serverens IP - så nginx sender forespørgsler videre til solr. Tjek også, at din forespørgsel ikke bliver sendt til den samme server hver gang. Det kan du fx gøre ved at trykke på core selector i venstre side - Her kan du se hvilke shards og replikas der ligger på den server du har ramt

## Indlæs data
For at kunne teste performance er vi nødt til at have noget data i solr, så den kommer på arbejde. Det data har vi i filen citites1000_fixed.txt

I kan indlæse det ved at køre følgende kommando i terminalen på jeres egen computer (sørg for at i står i samme mappe som ctities filen

```
curl -X POST "http://[IP]:8983/solr/[collection]/update?commit=true&separator=%09&fieldnames=id,name,,alternative_names,latitude,longitude,,,countrycode,,,,,,population,elevation,,timezone,lastupdate&overwrite=true" --data-binary @cities1000_fixed.txt  -H 'Content-type:application/csv'
```

Gå ind i solr UIet, og tjek at dokumenterne er blevet indlæst. Vælg navnet på jeres collection i venstre side og gå ind på query. Når i søger skulle i gerne få `numFound: 128766`.

## Test performance
Kør nogle queries, og udvælg dig en du gerne vil teste. Brug chromes inspector til at finde URL'en. Det burde være noget i stil med ```http://[IP]:8983/solr/cities/select?_=1539199282537&q=...``` 

Brug siege, apache bench eller noget helt tredje til at sammenligne hvordan solr performer når i sender queries direkte til den ene server, sammenlignet med at sende queries til load balanceren.
