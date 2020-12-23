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




## PREGUNTA 3 I 4: Anàlisi del conjunt de dades

* **Pregunta 3: Escull les variables que consideris dins el teu conjunt de dades per tal de descriure de manera completa una variable numèrica, una variable categòrica i les relacions numèrica-numèrica, numèrica-categòrica i categòrica-categòrica.**

Per fer aquest exercici, necessitarem algunes llibreries com ggplot2 o dplyr per exemplificar les relacions de manera il·lustrativa.

<ul>

Les dues variables principals són les següents:

  - **Variable numèrica:** Per una banda, tenim que una variable numèrica és tota variable quantitativa, és a dir, que a partir d'aquesta podem calcular infinits valors reals segons el que se'ns demana. Uns exemples ben clars a la pràctica que estem fent són les columnes: y, StringencyIndex, StringencyLegacyIndex, GovernmentResponseIndex, ContainmentHealthIndex, etc. (-0.05184844, 78.70, 76.19, 61.90, 72.22, 0.0, etc.).
 
  - **Variable categòrica:** Per altra banda, tenim que les variables categòriques són aquelles variables que no són quantitatives sinó qualitatives, és a dir, són aquelles que responen les preguntes amb respostes no-numèriques. Alguns dels exemples més clars a la pràctica serien el cas de les columnes: iso_code o country, ja que fan referència a les sigles del país d'on s'han enregistrat les dades o el nom d'aquest mateix. Ara bé, una variable categòrica també podria ser aquella que ens diu si una persona és de sexe masculí o femení, que ens diu si està casat o no (sí o no), aquelles que ens diuen quin curs o quins estudis està cursant la persona (primaria, secundaria, batxillerat, universitat, laboral...).
    
    
    
I com a relacions d'aquestes dues variables que acabem de veure, tenim:
   - **Relació numèrica-numèrica:** Parlem d'una relació numèrica quan dues variables són quantitatives i es poden fer una sèrie d'operacions que ens donaran una xifra que les relacionarà. Un exemple ben clar d'aquesta relació fent servir la taula que ens brinden seria.
   - **Relació numèrica-categòrica:** Aquesta és una mica més diferent en relació a la primera ja que es tracta de relacionar una variable quantitativa amb una qualitativa. Sol ser una relació entre una població concreta i alguna xifra rellevant que ens permetrà saber (dins un conjunt) en quina posició es troba. 
Nosaltres posem com exemple la relació de casos de covid per a cada continent. Els casos és una variable numèrica i els noms dels continents són la variable categòrica, d'aquesta manera podem visualitzar, aproximadament, quin és el nombre de casos respecte la resta de continents.

Aquest seria el codi que hem utilitzat per a construir el gràfic:
   
            library(ggplot2)
    ggplot(data, aes(x=ConfirmedCases, y=continent)) + geom_bar(stat="identity") +      scale_fill_brewer(palette="Oranges")
  
  
I aquesta és la imatge del gràfic:

  ![](https://github.com/estelbosch100/Treball-estadistica/blob/main/continent-confirmedcases.png)
     
Per exemple, podem estimar que a Oceania hi ha pocs casos en relació a la resta. A Oceania hi ha 7320 casos de covid, en canvi, a Àsia ja boregen els 1556201. Una dada molt distintiva és que a Nord Amèrica ja pràcticament arriben als 2500000 casos de covid.
  
   - **Relació categòrica-categòrica:** En aquest cas, se sol relacionar dues variables que, per exemple, donem per cas que parlem de persones que estan al món laboral. Aleshores, la primera variable descriu el sexe de la persona (masculí i femení) i la segona ens dona la opció de triar entre si el salari mensual és menor que 300, és menor que 1000 però major que 300 i si el salari mensual és major que 1000. Clar que això no seria un exemple aplicat a la pràctica. Per exemplificar aquesta relació tenint en compte la taula que se'ns dona, podríem afirmar que (ACABAR D'EMPLENAR I EXEMPLIFICAR AMB UN GRÀFIC)

</ul>


* **Pregunta 4: Utilitza els contrastos vistos a classes per treure conclusions o assumpcions que es poden fer amb les tres relacions de l’apartat anterior (numèrica-numèrica, numèrica-categòrica i categòrica-categòrica). Per cada un dels contrastos, específica les hipòtesis inicial i alternativa, quin és l’estadístic de contrast, el valor p i la decisió final envers les hipòtesis plantejades.**


## PREGUNTA 5 I 6: Model predictiu lineal

* **Pregunta 5: Crea un model de regressió lineal que expliqui la variable _y_ a partir de les variables d’acció de govern o altres variables derivades d’aquestes. Tria el model de manera que sigui òptim segons algun criteri vist a classe. Per aquest model, descriu l’efecte que tenen les variables explicatives amb la variable resposta i mira si es compleixen les assumpcions bàsiques.**

* **Pregunta 6: Utilitza aquest model per fer una predicció mitjana amb interval de confiança de la variable y al dia 15 de novembre, disposant únicament de les variables del conjunt _doxcgrt_.**
