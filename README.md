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
     
En aquest cas, primer es crea una nova taula Y on hi ha les files coincidents (entre la taula Y_b07 i Y_f07) i les columnes d'aquestes, obtingudes amb la intersecció de les taules i a partir del camp comú iso_code. La segona funció fa que creem una nova variable on es calculi el logaritme de casos entre les dues taules, que probablement més endavant s'hagin de fer servir.

Return only the rows in which the left table have matching keys in the right table.
* **Pregunta 2: Prepara el conjunt de dades _data_ per a ser analitzat segons les característiques de les variables. Explica les decisions que prens i quina és la tipologia de les variables que té el teu conjunt de dades.**

## PREGUNTA 3 I 4: Anàlisi del conjunt de dades

* **Pregunta 3: Escull les variables que consideris dins el teu conjunt de dades per tal de descriure de manera completa una variable numèrica, una variable categòrica i les relacions numèrica-numèrica, numèrica-categòrica i categòrica-categòrica.**

* **Pregunta 4: Utilitza els contrastos vistos a classes per treure conclusions o assumpcions que es poden fer amb les tres relacions de l’apartat anterior (numèrica-numèrica, numèrica-categòrica i categòrica-categòrica). Per cada un dels contrastos, específica les hipòtesis inicial i alternativa, quin és l’estadístic de contrast, el valor p i la decisió final envers les hipòtesis plantejades.**

## PREGUNTA 5 I 6: Model predictiu lineal

* **Pregunta 5: Crea un model de regressió lineal que expliqui la variable _y_ a partir de les variables d’acció de govern o altres variables derivades d’aquestes. Tria el model de manera que sigui òptim segons algun criteri vist a classe. Per aquest model, descriu l’efecte que tenen les variables explicatives amb la variable resposta i mira si es compleixen les assumpcions bàsiques.**

* **Pregunta 6: Utilitza aquest model per fer una predicció mitjana amb interval de confiança de la variable y al dia 15 de novembre, disposant únicament de les variables del conjunt _doxcgrt_.**
