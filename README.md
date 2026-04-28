Kompleksowy Raport Techniczny: Office Infiltration - Heavy Arsenal
Poniższy dokument stanowi całościową analizę architektury, mechaniki oraz matematycznych fundamentów gry. Projekt został zaprojektowany jako hybryda dynamicznej platformówki akcji z systemem „infinite runner”, gdzie progresja gracza jest definiowana przez dystans, zarządzanie ekonomią i strategiczne zakupy w punktach handlowych.

1. Wizja Projektu i Pętla Rozgrywki (Gameplay Loop)
Głównym założeniem Office Infiltration było stworzenie doświadczenia typu „easy to learn, hard to master”. Gra opiera się na ciągłym ruchu w prawą stronę ekranu, wymuszając na graczu balansowanie między agresywną eliminacją wrogów a unikaniem obrażeń.
Rdzeń rozgrywki (Core Loop):
1.	Eksploracja i Walka: Pokonywanie kolejnych metrów biurowca, eliminacja patroli i zbieranie wypadających monet.
2.	Ekonomia: Gromadzenie kapitału z pokonanych wrogów (różne stawki za różne typy jednostek).
3.	Progresja w Sklepie: Docieranie do terminali „Black Market”, gdzie gracz musi podejmować decyzje: czy zainwestować w natychmiastowe leczenie, czy oszczędzać na potężniejszą broń lub permanentne zwiększenie paska zdrowia.
4.	Skalowanie Trudności: Wraz ze wzrostem dystansu, gra modyfikuje tablicę prawdopodobieństwa spawnu przeciwników, wprowadzając jednostki opancerzone.









2. Zaawansowana Architektura Fizyki i Kolizji
Silnik gry operuje na autorskim systemie fizyki 2D, który rezygnuje z gotowych bibliotek na rzecz czystego kodu JavaScript i matematyki wektorowej.
•	Model Ruchu i Grawitacji: Zastosowano system stałej akceleracji grawitacyjnej ($g = 0.95$) oraz tarcia ($friction = 0.85$). Każda klatka animacji przelicza wektor prędkości $v_x$ i $v_y$. Tarcie sprawia, że postać nie zatrzymuje się w miejscu, lecz płynnie wyhamowuje, co nadaje ruchowi ciężaru i realizmu.
•	System Kolizji AABB (Axis-Aligned Bounding Box): Detekcja kolizji odbywa się poprzez porównywanie krawędzi prostokątów obiektów. Kluczowym rozwiązaniem jest rozróżnienie typów podłoża:
•	Floor (Podłoga): Nieprzenikalna bariera, która resetuje jumps do wartości 0.
•	Desk (Biurko): Platforma jednostronna. Logika player.y + player.h < p.y + p.h + player.velY pozwala graczowi wskakiwać na biurka od dołu, ale blokuje go podczas opadania. Dodatkowa obsługa klawisza S pozwala „przebić się” przez tę warstwę, co dodaje głębi taktycznej podczas ucieczki przed pociskami.
•	Kamera Progresywna: Kamera nie jest statyczna ani sztywno przypisana do środka. Obliczanie cameraX = player.x - canvas.width / 3 sprawia, że gracz zawsze widzi więcej terenu przed sobą niż za sobą, co jest kluczowe w dynamicznych strzelankach.





3. Mechanika Balistyki i Systemu Walki
System walki został zaprojektowany z myślą o różnorodności doświadczeń płynących z używania różnych narzędzi mordu.
•	Słownik Broni (Weapon Dictionary): Każda broń w obiekcie WEAPONS posiada unikalną sygnaturę. Pistolet oferuje precyzję, strzelba generuje 10 niezależnych pocisków z dużym rozrzutem (spread), a karabin (Rifle) stawia na dużą szybkostrzelność kosztem celności.
•	Matematyka Rozrzutu i Balistyki: Podczas strzału gra oblicza kąt między graczem a kursorem myszy za pomocą funkcji Math.atan2. W przypadku broni o dużym rozrzucie, do końcowego kąta dodawana jest wartość losowa:
const spread = (Math.random() - 0.5) * currentWeapon.spread;
Zapewnia to organiczne zachowanie pocisków, które nie lecą idealnie w jeden punkt.
•	System Eksplozji (AOE): Wprowadzenie granatnika zmieniło paradygmat walki z punktowej na obszarową. Eksplozje posiadają własny cykl życia (timer). Obrażenia są zadawane tylko w pierwszej klatce wybuchu, co zapobiega błędnemu naliczaniu obrażeń (tzw. „damage stacking”) przy każdej klatce animacji płomieni.

4. Proceduralne Generowanie Świata i AI
Gra nie posiada predefiniowanych poziomów. Każda rozgrywka jest unikalna dzięki algorytmowi generateWorld.
•	Generowanie „Just-In-Time”: Nowe segmenty poziomu są tworzone, gdy gracz zbliży się do krawędzi wygenerowanego świata (lastGeneratedX < player.x + 3000). Zapobiega to obciążeniu pamięci przez obiekty, których gracz jeszcze nie widzi.
•	Ewolucyjne AI: Przeciwnicy nie tylko patrolują teren, ale reagują na odległość gracza. Każdy typ wroga ma zdefiniowany parametr burst (liczba strzałów w serii) oraz cooldown.
•	Lekkie jednostki: Strzelają pojedynczo, łatwe do wymanewrowania.
•	Jednostki Juggernaut: Posiadają zmodyfikowany model renderowania (skalowanie szerokości i wysokości) oraz ogromną pulę HP. Ich obecność na późnych etapach gry wymusza na graczu posiadanie broni o wysokim DPS (Damage Per Second).

5. Ekonomia i Sklep (Black Market)
System ekonomiczny pełni rolę mechanizmu „risk vs reward”.
•	Fizyka Łupu: Monety po śmierci wroga nie pojawiają się statycznie. Posiadają własny wektor startowy vx i vy, co sprawia, że rozlatują się na boki i odbijają od platform. Gracz musi często ryzykować wystawienie się na ogień, aby zebrać fundusze przed ich zniknięciem lub zablokowaniem.
•	Struktura Handlu: Sklepy pojawiają się w regularnych interwałach (co 10,000 jednostek dystansu). Każdy sklep oferuje:
•	Naprawę (Health): Stały koszt, odnawia część HP.
•	Ulepszenie Pancerza (Max HP): Zwiększa limit zdrowia, co jest niezbędne, by przetrwać trafienie z granatnika wrogiego Grenadiera.
•	Nową Broń: Losowo wybrany model, który całkowicie zmienia styl gry.

6. System Debugowania i Secret Room
W kodzie zaszyto potężne narzędzie deweloperskie pod klawiszem \. Przenosi ono gracza do odizolowanej strefy testowej (Secret Room).
•	Spawners: Specjalne interaktywne obiekty pozwalające na natychmiastowe przywołanie dowolnego typu przeciwnika (nawet Juggernauta) w celu przetestowania balansu obrażeń.
•	Weapon Pickups: Możliwość darmowego przełączania się między wszystkimi typami broni bez konieczności zbierania monet.
•	Izolacja Świata: Strefa ta korzysta z osobnej tablicy platform secretPlatforms, co udowadnia elastyczność silnika kolizji, który może przełączać się między różnymi zestawami danych środowiskowych w czasie rzeczywistym.









7. Specyfikacja Techniczna Jednostek
Klasa	Punkty Życia (HP)	Typ Ataku	Nagroda ($)	Charakterystyka
Pistol Guard	100	Pojedynczy	30	Podstawowy wróg, niska celność.
Elite Rifleman	80	Serie po 6	70	Szybki, niski profil HP, groźny w grupach.
Grenadier	250	Granat AOE	70	Powolny, ale potrafi zabić jednym trafieniem.
Juggernaut	500	Ciężki karabin	200	Opancerzony gigant, wysoka szybkostrzelność.
Juggernaut-S	500	Salwa Shotguna	200	Ekstremalnie niebezpieczny na bliskim dystansie.

