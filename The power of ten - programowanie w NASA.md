Używane też w jet propulsion laboratory w Kalifornijskim Instytucie Technologii (w kontrakcie z NASA)

1. Ograniczać kod do control flow constructs - brak goto, setjmp, longjmp, rekurencji
	1. łatwiejsza weryfikacja
	2. zwiększona klarowność kodu
	3. brak rekurencji - wszystkie wykonania, które powinny mieć zakres, faktycznie go mają
2. Wszystkie pętle muszą mieć odgórne ograniczenie
	1. narzędzie do analizy musi umieć statycznie wykazać, że pętla zatrzyma się po jakiejś ilości iteracji
	2. chyba, że pętla nie ma się zatrzymywać - wtedy narzędzie musi wykazać, że pętla się nie zatrzyma
	3. kiedy górny limit jest osiągnięty, assertion failure się odpala, a funkcja, w której zawiera się pętla zwraca błąd
3. nie korzysta się ze sterty (z dynamicznej alokacji pamięci)
	1. malloc i garbage collectory mają nieprzewidywalne zachowania, które spowalniają program
	2. programiści mogą wpaść w pułapkę niezwalniania pamięci lub korzystania z pamięci, które została zwolniona, itd itd
	3. skoro nie ma rekurencji, to da się statycznie wykazać ile program weźmie pamięci stosowej i da się udowodnić, że aplikacja będzie działać tylko w prealokowanej pamięci
4. funkcja nie dłuższa niż to, co zmieści się na kartce papieru (nie więcej niż około 60 linijek kodu)
	1. każda funkcja powinna być logiczną jednostką, która jest rozumiana i weryfikowalna jako jednostka 
5. minimum dwa warunki sprawdzające (assertions)
	1. te warunki nie mogą mieć efektów ubocznych
	2. muszą być zdefiniowane jako testy boole'owskie
	3. jeżeli assertion nie jest spełnione, to musi zwrócić błąd do wywoławcy, którego wykonanie sprawiło, że assertion zostało striggerowane
6. dane muszą być zadeklarowane w najniższym możliwym zakresie; ograniczenie "uglobalizowania" danych do minimum
	1. jeśli nie jest w zakresie, to nie da się tego zepsuć ani do tego przez przypadek odnieść
	2. jeżeli jakiś obiekt wywołuje błąd, to łatwiej odnaleźć ten błąd, jeśli zakres poszukiwań jest zmniejszony
7. to, co zwracają funkcje nie-voidowe, musi być sprawdzone przez każdą funkcję wywołującą oraz parametry funkcji muszą być zwalidowane wewnątrz tej funkcji 
	1. jeżeli to co zwraca funkcja jest nieistotne to trzeba opisać to i uzasadnić czemu jest nieistotne; tak samo kiedy zrobi się cast na void w zwróceniu
8. użycie preprocesora musi być ograniczone do zawarcia plików nagłówkowych i zdefiniowania prostych definicji makr. wszystkie makro muszą rozwijać się do jednostek składniowych. Rzadko kiedy jest uzasadnienie więcej niż dwóch makr. Uwaga na ostrożność przeciwko kompilacji warunkowej jest uzasadniona tym, że wraz ze wzrostem ilości warunków kompilacji ilość różnych wersji kodu rośnie wykładniczo - przy 10 warunkach jest 2^10 możliwych wersji kodu  
9. użycie wskaźników powinno być ograniczone, szczególnie zwraca się uwagę na wskaźniki, które można zdereferencjować więcej niż raz; nie wolno korzystać ze wskaźników na funkcje
	1. łatwo ich źle użyć
	2. trudniej analizuje się przepływ danych w programie
	3. wskaźniki na funkcje - narzędzia do statycznej analizy kodu mają z nimi problem. Przykład:
		1. narzędzie do statycznej analizy nie ma jak sprawdzić, czy nie występuje rekurencja, zatem kiedy już naprawdę trzeba skorzystać ze wskaźnika na funkcję, to trzeba też dać alternatywną gwarancję braku rekurencji
10. cały kod musi być kompilowany od początku oraz w jak najbardziej pedantycznych ustawieniach kompilatora, cały kod musi kompilować się bez błędów. Codziennie trzeba analizować kod wykorzystując narzędzia do analizy statycznej; ta analiza nie może zwrócić błędu
