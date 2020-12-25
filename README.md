# TREBALL FINAL D'ESTADÍSTICA
_Gabriel Romero i Estel Bosch - U1966628 i U1966682_

## PREGUNTA 1 I 2: Processament del conjunt de dades a analitzar


* **Pregunta 1: Comenta en detall cada un dels següents chunks utilitzats pel processament de dades:**


      Y_b07 = filter(dowid, DAY - days(7) < date, date <= DAY): 
El què ens fa aquesta comanda és filtrar i mostrar totes les dades recopilades en un dia concret (utilitzant el codi U1966628 obtenim la data 13/06), en el nostre cas, el 13 de Juny.

     Y_b07 = group_by(Y_b07, iso_code, country = location) i Y_b07 = summarise(Y_b07, .groups = 'drop', cases_b7 = mean(new_cases, na.rm = TRUE)):
    
Aquestes dues comandes, el què fan és agrupar-nos les dades en tres variables que són: Les sigles del país, els diferents països dels quals s'ha enregistrat les dades en un dia concret i aquest mateix dia. (Y_b07: dia, iso_code: sigles del país, country: país).

     Y_b07 = filter(Y_b07, cases_b7 > 0): 
    
 Ens ha acotat tots els països que tenen casos de covid.
   
    Y_f07 = dowid %>%
    filter(DAY < date, date <= DAY + days(7)) %>%
    group_by(iso_code) %>%
    summarise(.groups = 'drop', cases_f7 = mean(new_cases, na.rm = TRUE)) %>%
    filter(cases_f7 > 0):
  
Crea un nou data_frame que es diu Y_f07 a partir de la taula que ja teníem construïda. Aquest nou data frame es crea gràcies a la crida de diferents funcions com: la filtració del dia del qual es volen les dades, l'agrupació o group by de dades segons les sigles del país en què s'han observat, amb summarise podem obtenir la mitjana dels nous casos de covid i, finalment, a aquesta última funció se'n filtren els casos perquè siguin estrictement majors a 0 casos de covid.
    
     Y = inner_join(Y_b07, Y_f07, by = "iso_code") %>%
     mutate(y = log(cases_f7/cases_b7))
     
En aquest cas, primer es crea una nova taula "Y" on hi ha les files coincidents (entre la taula Y_b07 i Y_f07) i les columnes d'aquestes, obtingudes amb la intersecció de les taules i amb l'ajuda del camp comú "iso_code" entre ambdues. La segona funció fa que creem una nova variable on es calculi el logaritme de casos entre les dues taules, que probablement més endavant s'hagin de fer servir.

      temp = filter(doxcgrt, is.na(RegionName), ymd(Date) == DAY)
Una nova variable anomenada "temp" on es filtren els valors de la taula _doxcgrt_ amb la data DAY i que el nom de regió sigui _NA_.


      X = select(temp, iso_code = CountryCode, ends_with("Index"), matches('^(C|E)._'), -ends_with('_Flag')) %>%
      rename_with(~str_sub(., 1, 2), matches('^(C|E)._'))
Utilitzant la llibreria _stringr_ es crea una taula X on hi ha les columnes "iso_code" equivalent a les sigles dels països, tal i com havíem dit anteriorment, totes aquelles que acaben amb la paraula "Index" com, per exemple, "StringencyIndex", "GovernmentResponseIndex", ...; 
A més a més, hi haurà totes aquelles columnes que es diessin **CN_Flag** o **EN_Flag**, on N representa un dígit. Aquestes columnes passaran a dir-se C1,C2,C3,..., o E1,E2,E3... segons el prefix que tinguessin.

      data = inner_join(Y, X, by = 'iso_code')
      
Per últim, creem una última taula referent a la primera pregunta del treball i que es diu _data_.
Aquesta l'hem obtingut a partir de l'intersecció entre les taules _X_ i _Y_ que hem creat als apartats anteriors. Òbviament, al ser una intersecció, han de tenir un camp en comú i el que hem fet servir ha sigut l'"iso_code" també vist en apartats previs.

#### Estem interessats a analitzar quina relació hi ha entre les decisions de govern (polítiques de tancament i contenció i econòmiques) i com es relacionen aquestes amb la variació de casos a la població (y). Tenint en compte això, contesta les preguntes següents del treball:

* **Pregunta 2: Prepara el conjunt de dades _data_ per a ser analitzat segons les característiques de les variables. Explica les decisions que prens i quina és la tipologia de les variables que té el teu conjunt de dades.**

Per fer l'exercici 2 cal observar amb atenció la resta d'exercicis ja que se'ns demana modificar la taula data i això només es pot fer a partir dels exercicis a continuació. 

Primer de tot, hem mirat si totes les files de la taula tenen dades, n'hi ha una que no. Aquesta que correspon a Comoros amb iso_code = COM la desestimem perquè si no s'han pogut obtenir dades, no ens pot servir per a calcular cap altre valor.

En segon lloc, hem mirat les relacions que cal fer entre algunes variables de la taula. Com que hem de fer una relació numèrica-numèrica, una numèrica-categòrica i una categòrica-categòrica, creiem convenient que haurem de crear noves columnes que ens permetin obtenir aquestes relacions. Ens ha semblat interessant ajuntar les variables _ConfirmedCases_ i _ConfirmedDeaths_ de la taula temp amb data ja que són dades que ens poden ajudar a calcular un exemple de relació numèrica-numèrica. 
      
      auxiliar = select(temp, iso_code = CountryCode, ConfirmedCases, ConfirmedDeaths)
      data = inner_join(data, auxiliar, by='iso_code')
      
 Seguidament, també hem volgut inserir alguna columna més de la taula dowid a data i hem eliminat els valors duplicats fent ús de la funció distinct() de la llibreria _dplyr_ : 
 
 
      auxiliar = select(dowid, iso_code, continent, median_age, aged_65_older, aged_70_older)
      data = inner_join(data, auxiliar, by = 'iso_code')

      data = distinct(data)


Creiem convenient crear una _taula auxiliar_ on hi posarem aquelles columnes que ens interessen de la taula _data_ per a veure més clar sobre quines dades treballem i quines totalitats tenim entre les mans. A continuació, filtrarem aquesta taula per continents (Àsia, Àfrica, Amèrica del Nord, Amèrica del Sud, Europa i Oceania):
            
            TaulaAux = select(data, continent, ConfirmedCases, ConfirmedDeaths)

Filtratge per continent:
            
            africa_casos = filter(TaulaAux, continent == "Africa")
            asia_casos = filter(TaulaAux, continent == "Asia")
            northamerica_casos = filter(TaulaAux, continent == "North America")
            southamerica_casos = filter(TaulaAux, continent == "South America")
            europa_casos = filter(TaulaAux, continent == "Europe")
            oceania_casos = filter(TaulaAux, continent == "Oceania")

 Ara tenim 6 taules corresponents a cada continent, per fer-ho més visual, tenim aquestes dades a la taula _africa_casos_ :
 
            
![](C:\Users\estel\Documents\GeinfUdg\2n CURS\ESTADÍSTICA\PRÀCTIQUES\Treball estadistica\AfricaCasos.png)


Ara voldríem fer el sumatori per a cada columna _ConfirmedCases_ i _ConfirmedDeaths_ per tenir sobre quin total tenim les dades:

_ConfirmedCases_ :

            CasosAfrica = sum(africa_casos$ConfirmedCases)
            CasosAsia = sum(asia_casos$ConfirmedCases)
            CasosNordAmerica = sum(northamerica_casos$ConfirmedCases)
            CasosSudAmerica = sum(southamerica_casos$ConfirmedCases)
            CasosEuropa = sum(europa_casos$ConfirmedCases)
            CasosOceania = sum(oceania_casos$ConfirmedCases)
      
I els valors que ara tenim en els casos de covid per a cada continent són: 
            
            CasosAfrica = 228638
            CasosAsia = 1556201
            CasosNordAmerica = 2385773
            CasosSudAmerica = 1421503
            CasosEuropa = 2147099
            CasosOceania = 7320
      
_ConfirmedDeaths_ :

            DeathsAfrica = sum(africa_casos$ConfirmedDeaths)
            DeathsAsia = sum(asia_casos$ConfirmedDeaths)
            DeathsNordAmerica = sum(northamerica_casos$ConfirmedDeaths)
            DeathsSudAmerica = sum(southamerica_casos$ConfirmedDeaths)
            DeathsEuropa = sum(europa_casos$ConfirmedDeaths)
            DeathsOceania = sum(oceania_casos$ConfirmedDeaths)

I els valors que ara tenim en els casos de mort per covid de cada continent són: 
            
            DeathsAfrica = 6183
            DeathsAsia = 39176
            DeathsNordAmerica = 145366
            DeathsSudAmerica = 59068
            DeathsEuropa = 180113
            DeathsOceania = 102
            


## PREGUNTA 3 I 4: Anàlisi del conjunt de dades

* **Pregunta 3: Escull les variables que consideris dins el teu conjunt de dades per tal de descriure de manera completa una variable numèrica, una variable categòrica i les relacions numèrica-numèrica, numèrica-categòrica i categòrica-categòrica.**

Per fer aquest exercici, necessitarem algunes llibreries com ggplot2 o dplyr per exemplificar les relacions de manera il·lustrativa.

<ul>

Les dues variables principals són les següents:

  - **Variable numèrica:** Per una banda, tenim que una variable numèrica és tota variable quantitativa, és a dir, que a partir d'aquesta podem calcular infinits valors reals segons el que se'ns demana. Uns exemples ben clars a la pràctica que estem fent són les columnes: y, StringencyIndex, StringencyLegacyIndex, GovernmentResponseIndex, ContainmentHealthIndex, etc. (-0.05184844, 78.70, 76.19, 61.90, 72.22, 0.0, etc.).
 
  - **Variable categòrica:** Per altra banda, tenim que les variables categòriques són aquelles variables que no són quantitatives sinó qualitatives, és a dir, són aquelles que responen les preguntes amb respostes no-numèriques. Alguns dels exemples més clars a la pràctica serien el cas de les columnes: iso_code o country, ja que fan referència a les sigles del país d'on s'han enregistrat les dades o el nom d'aquest mateix. Ara bé, una variable categòrica també podria ser aquella que ens diu si una persona és de sexe masculí o femení, que ens diu si està casat o no (sí o no), aquelles que ens diuen quin curs o quins estudis està cursant la persona (primaria, secundaria, batxillerat, universitat, laboral...).
    
    
    
I com a relacions d'aquestes dues variables que acabem de veure, tenim:
   - **Relació numèrica-numèrica:** Parlem d'una relació numèrica quan dues variables són quantitatives i es poden fer una sèrie d'operacions que ens donaran una xifra que les relacionarà. 
Començarem seleccionant de data les columnes que ens interessen per a calcular el percentatge de mortalitat segons el registre de casos covid a cada continent. Per diferenciar-ho de la relació numèrica-categòrica que veurem a continuació, el què farem serà tractar cada continent per separat, obtenint uns valors independents.


Així doncs, per mostrar visualment com és un exemple de relació numèrica-nuèrica, seria fer servir la freqüència de dades recopilades sobre el nivell de rigorositat près a cadascuna d'aquestes. Per fer-ho, utilitzarem un histograma:

    
    hist(x = data$StringencyIndex, main = "Histograma de la rigorositat presa pels governs", 
         xlab = "Nivell de rigorositat", ylab = "Frequencia",
         col = "purple")

  ![](http://127.0.0.1:28001/graphics/d8032a61-02f1-4d2b-97f2-f49480c47bd0.png)
   
   - **Relació numèrica-categòrica:** Aquesta és una mica més diferent en relació a la primera ja que es tracta de relacionar una variable quantitativa amb una qualitativa. Sol ser una relació entre una població concreta i alguna xifra rellevant que ens permetrà saber (dins un conjunt) en quina posició es troba. 
Nosaltres posem com exemple la relació de casos de covid per a cada continent. Els casos és una variable numèrica i els noms dels continents són la variable categòrica, d'aquesta manera podem visualitzar, aproximadament, quin és el nombre de casos respecte la resta de continents.

Aquest seria el codi que hem utilitzat per a construir el gràfic:
   
            library(ggplot2)
    ggplot(data, aes(x=ConfirmedCases, y=continent)) + geom_bar(stat="identity") +      scale_fill_brewer(palette="Oranges")
  
  
I aquesta és la imatge del gràfic:
![Casos confirmats de covid](http://127.0.0.1:28001/graphics/a0e8ddfa-ede4-45dc-86a1-64e06f7d4973.png)


Per exemple, podem estimar que a Oceania hi ha pocs casos en relació a la resta. A Oceania hi ha 7320 casos de covid, en canvi, a Àsia ja boregen els 1556201. Una dada molt distintiva és que a Nord Amèrica ja pràcticament arriben als 2500000 casos de covid.
  
   - **Relació categòrica-categòrica:** En aquest cas, se sol relacionar dues variables que, per exemple, donem per cas que parlem de persones que estan al món laboral. Aleshores, la primera variable descriu el sexe de la persona (masculí i femení) i la segona ens dona la opció de triar entre si el salari mensual és menor que 300, és menor que 1000 però major que 300 i si el salari mensual és major que 1000. Clar que això no seria un exemple aplicat a la pràctica. Caldria fer una taula de contingències sobre l'ajut econòmic que reben els diferents continents.
   
El què fem és crear una taula _Economia_ auxiliar per tal de poder agafar les dades de la taula Data que siguin de l'edat mitjana i continent:

            
      Economia = select(data, continent, EconomicSupportIndex)
      Economia <- Economia[!is.na(Economia$EconomicSupportIndex),]

            
Seguidament, el què fem és construir una taula de contingència amb aquestes dades:
      
      table(Economia$continent, Economia$EconomicSupportIndex)
      
El resultat serà una taula bastant gran on s'hi veurà a l'esquerra el nom del continent i a sobre podrem veure diferents ajuts econòmics donats als continents.

![]() hi va una imatge


</ul>


* **Pregunta 4: Utilitza els contrastos vistos a classes per treure conclusions o assumpcions que es poden fer amb les tres relacions de l’apartat anterior (numèrica-numèrica, numèrica-categòrica i categòrica-categòrica). Per cada un dels contrastos, específica les hipòtesis inicial i alternativa, quin és l’estadístic de contrast, el valor p i la decisió final envers les hipòtesis plantejades.**


## PREGUNTA 5 I 6: Model predictiu lineal

* **Pregunta 5: Crea un model de regressió lineal que expliqui la variable _y_ a partir de les variables d’acció de govern o altres variables derivades d’aquestes. Tria el model de manera que sigui òptim segons algun criteri vist a classe. Per aquest model, descriu l’efecte que tenen les variables explicatives amb la variable resposta i mira si es compleixen les assumpcions bàsiques.**

En primer lloc, com bé hem vist a les pràctiques d'estadística, el model lineal és el model que explica l’aleatorietat d’un variable numèrica y (coneguda també com a variable resposta) com a combinació lineal d’altres variables numèriques.


* **Pregunta 6: Utilitza aquest model per fer una predicció mitjana amb interval de confiança de la variable y al dia 15 de novembre, disposant únicament de les variables del conjunt _doxcgrt_.**
