# Dokumentace k úkolu č.2  - dělení adresních bodů

Tento program slouží rozdělení bodů do skupin/clusterů.

Příklad spuštění scriptu v příkazové řádce:
python.exe  du2.py  input.geojson  output.geojson  -mp 100


## Vstupní parametry
  * 1.parametr: název scriptu
  * 2.parametr: vstupní soubor
  * 3.parametr: název výstupního souboru
  * 4.parametr: maximální počet bodů v jednom clusteru (volitelný parametr, defaultní hodnota je 50)


## Výstup
Soubor ve formátu GeoJSON, jehož hodnota atributu 'cluster_id' určuje příslušnost bodu do clusteru.


## Popis funkcí

*get_args(string_value)*
 * funkce slouží k načtení parametrů programu, konkrétně: vstupní soubor, výstupní soubor a maximální počet prvků
    v jednom clusteru
 * funkce zároveň ověřuje korektnost zadání jednotlivých parametrů:
   - vstupní soubor (input_geojson):
     - ověří, zda soubor existuje a je možné jej otevřít (zda se jedná o korektní GeoJSON)
     - soubor otevře pro čtení
   - výstupní soubor(output_geojson):
     - vytvoří soubor použitelný k zápisu (otevře soubor pro psaní)
   - výstupní soubor(output_geojson):
     - volitelný parametr, který se do příkazové řádky zapisuje ve formátu: -mp <hodnota>
     - pokud není při spuštění programu definován, nastaví se jeho hodnota na 50
     - ověření jeho správnosti viz fce *int_gt_1()*
 * u každého parametru je krátká vysvětlivka pro užití v helpu programu
 * funkce vrátí parametry/proměnné programu


*int_gt_1()*
 * funkce slouží k ověření validity vstupu volitelného počtu bodů v clusteru
 * vstupní hodnota je převedena na datový typ *int*
 * ověření, zda je zadaná hodnota větší nebo rovna 1 (jinak by dělení nedávalo smysl)


*calculate_bbox(features)*
 * funkce určí minimální a maximální hodnoty souřadnic vstupních bodů na osách x a y, tedy poskytne souřadnice
 ohraničujícího obdélníku
 
 
*get_half_value(min_value, max_value)*
 * funkce určí osy stran ohraničujícího obdélníku bodů


*sort_features(features, half_x, half_y)*
 * funkce rozdělí vstupní data do skupin/listů dle příslušnosti do kvadrantu a přiřadí jim příslušné id clusteru
 * body jsou do jednotlivých kvadrantů rozřazeny na základě porovnávání souřadnic bodů a os
 * principt tvorby (hodnoty) 'cluster_id':
   - ohraničující obdélník vstupních bodů je rozdělen na "pomyslné" kvadranty s následujícím označením:
	 1 - levý horní kvadrant
	 2 - pravý horní kvadrant
	 3 - levý dolní kvadrant
	 4 - pravý dolní kvadrant
   - v případě, kdy se bod nachází na dělící ose, je přiřazen do kvadranty " na pravo" od osy ve směru hodinových
   ručiček
   - v případě, kdy se bod nachází na průsečíku dělících os, je přiřazen do prvního kvadrantu
   - při rozřazování bodů se vždy přidá hodnota kvadrantu, do nějž byl bod přiřazen(při opakovaném volání funkce tedy
     dochází k "řetězení" hodnoty cluster_id)
   - bod s delší hodnotou 'cluster_id', byl rozřazován vícekrát a náleží tak do (prostorově)menšího clusteru
   - např. bod s 'cluster_id' = 1 byl vytvořen v prvním průběhu funkce a nachází se v pomyslném levém horním kvadrantu,
   bod s 'cluster_id' = 23 byl při prvním průběhu funkce určen v pravém horním kvadrantu a při druhém průběhu funkce,
   kdy byl dělen kvadrant s označením 2 byl určen do levého dolního rohu kvadrantu s označením 2, proto má tedy 
   hodnotu 23 ('2' byla připsána při prvním průběhu/dělení a '3' při druhém průběhu/dělení bodů)


*quadtree(input_features, output_json, max_features, min_x, min_y, max_x, max_y)*
 * rekurzivní funkce, která rozdělí vstupní body do clusterů tak, aby byla splněna podmínka maximálního počtu bodů v 
 jednom clusteru
 * dochází k opakovanému dělení kvadrantů na kvadranty (atd.), dokud se v daném kvadrantu/clusteru nenachází menší počet
  objektů, než je definováno
 * samotný průběh funkce se skládá z výpočtu os (get_half_value), určení příslušnosti bodu do kvadrantu (sort_features),
  a opakovaného volání funkce pro konkrétní kvadranty, dokud není splněn požadovaný maximální počet bodů
 * v případě, že již není potřeba kvadrant/cluster dále dělit, se data zapíší do listu připraveného k zápisu
  do výstupního souboru

*run()*
 * funkce slouží ke spuštění jednotlivých funkcí ve správném pořadí a se správnými parametry
 * načtení parametrů/proměnných programu
 * načtení vstupních dat
 * ověření, zda se jedná o validní GeoJSON (zda soubor/slovník obsahuje klíče 'type' a 'features'
 * vytvoření/vyjmutí 'listu features' ze vstupních dat
 * předpřipravení výstupního souboru - zkopírování klíčů a hodnot vstupního souboru (s výjimkou klíče 'features' a jeho hodnot)
 * předpřipravení 'listu features' výstupního souboru (prázdný)
 * výpočet ohraničujícího obdélníku
 * přiřazení bodů do clusterů (volání funkce quadtree, která v sobe volá funkci sort_features)
 * zápis dat do výstupního souboru

## Popis testů

### Testy ověřující počet argumentů

*get_return_code(input_list)*
 * funkce slouží k získání hodnoty chybového kódu 
 * "pomůcka" k dalším testům
 * input_list - list parametrů programu (pro spuštění)

*test_no_argument()*
 * ověřuje, zda v případě, kdy není zadán žádný vstupní parametr, je hodnota chyby rovna 2

*test_nonexisting_input()*
 * ověřuje, zda v případě, kdy vstupní soubor neexistuje, je hodnota chyby rovna 2

*test_missing_output()*
 * ověřuje, zda v případě, kdy není zadán název výstupního souboru, je hodnota chyby rovna 2

*test_success()*
 * ověřuje, zda v případě, kdy je zadán vstupní a výstupní soubor, proběhne program bez chyby (chyba je rovna 0)

*test_invalid_mp()*
 * ověřuje, zda v případě, kdy je zadán maximální počet bodů v clusteru 0, je hodnota chyby rovna 2

*test_valid_mp()*
 * ověřuje, zda v případě, kdy je zadána validní hodnota maximálního počtu bodů, proběhne program bez chyby (chyba je rovna 0)

### Další testy

*test_valid_output_geojson()*
 * test ověřující validitu výstupního GeoJSONU
 * ověřuje, zda jsou ve výstupním souboru klíče 'type' a 'features'
 

*test_quadtree()*
 * test dělící funkce
 * ověřuje, zda se hodnota 'cluster_id' nevyskytuje vícekrát než je zadané maximum (v případě testu je to 50x)
 * vytvoření "kontrolního" slovníku, kde je klíčem hodnota 'cluster_id' a hodnotou tohoto klíče je počet výskytů






 

