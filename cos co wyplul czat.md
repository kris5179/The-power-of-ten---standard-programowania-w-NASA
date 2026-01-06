### Co to jest?

"The Power of Ten" to minimalistyczny standard kodowania, który drastycznie ogranicza swobodę programisty w języku C. W przeciwieństwie do obszernych standardów (jak MISRA C, który ma setki reguł), Holzmann postawił na 10 kluczowych zasad, które są łatwe do zapamiętania i – co najważniejsze – **umożliwiają automatyczną weryfikację poprawności kodu**.

Zasady te zmieniają język C z luźnego, "niebezpiecznego" narzędzia w bardziej deterministyczny podzbiór, który da się łatwiej analizować matematycznie.

### Dlaczego to ważne?

W inżynierii oprogramowania kosmicznego błędy są nieakceptowalne – nie można wysłać serwisanta na Marsa, by zrestartował system. Ważne są dwa aspekty:

1. **Bezpieczeństwo pamięci i wykonania:** Eliminacja wycieków pamięci, przepełnień stosu i nieprzewidywalnych zachowań.
    
2. **Weryfikowalność (Analyzability):** To kluczowy punkt dla Ciebie jako studenta. Większość błędów w kodzie jest trudna do wykrycia przez _statyczną analizę kodu_ (analizatory kodu), ponieważ przestrzeń stanów programu jest zbyt duża. "The Power of Ten" redukuje tę przestrzeń, czyniąc kod "przezroczystym" dla narzędzi analizujących.
    

### Fundamenty: Matematyka i weryfikacja formalna

Aby zrozumieć, dlaczego te reguły są tak restrykcyjne, musimy sięgnąć do teorii obliczeń. Fundamentalnym problemem jest **Problem Stopu** (_Halting Problem_).

Dla dowolnego programu $P$ i danych wejściowych $I$, nie istnieje algorytm, który w ogólnym przypadku rozstrzygnie, czy $P$ zatrzyma się dla $I$, czy wpadnie w pętlę nieskończoną. Matematycznie, funkcja decyzyjna jest nierozstrzygalna.

$$H(P, I) = \begin{cases} 1 & \text{jeśli } P(I) \text{ się zatrzymuje} \\ 0 & \text{w p.p.} \end{cases}$$

Narzędzia do statycznej analizy kodu próbują udowodnić poprawność programu bez jego uruchamiania. Jeśli kod jest zbyt skomplikowany (wskaźniki na wskaźniki, rekurencja), narzędzie musiałoby rozwiązać problem stopu, co jest niemożliwe. "The Power of Ten" narzuca ograniczenia, które sprawiają, że kod staje się **rozstrzygalny** dla analizatorów w rozsądnym czasie.

---

### Jak to działa "pod maską"? (Analiza 10 reguł)

Oto zasady wraz z wyjaśnieniem technicznym:

#### 1. Prosty przepływ sterowania

Zabronione jest używanie `goto`, `setjmp`, `longjmp` oraz **bezpośredniej i pośredniej rekurencji**.

- **Pod maską:** Rekurencja utrudnia wyliczenie maksymalnego zużycia stosu (_stack usage_). W systemach wbudowanych stos jest mały i sztywny. Eliminując rekurencję, możemy statycznie udowodnić, że **Stack Overflow** nigdy nie wystąpi, ponieważ graf wywołań funkcji staje się acykliczny.
    

#### 2. Sztywne granice pętli

Wszystkie pętle muszą mieć sztywny górny limit iteracji (ang. _fixed upper bound_).

- **Pod maską:** To bezpośrednia walka z problemem stopu. Jeśli każda pętla ma limit, możemy obliczyć **WCET** (_Worst-Case Execution Time_). Jest to niezbędne w systemach czasu rzeczywistego (RTOS), gdzie zadanie musi się wykonać w ściśle określonym oknie czasowym (np. 10 ms).
    

#### 3. Zakaz dynamicznej alokacji pamięci po inicjalizacji

Zabronione używanie `malloc`, `free`, `new` po starcie systemu.

- **Pod maską:** Dynamiczna alokacja prowadzi do fragmentacji sterty (_heap fragmentation_). W misji trwającej 10 lat fragmentacja może doprowadzić do sytuacji, w której jest wolna pamięć, ale nie ma ciągłego bloku dla nowej zmiennej. Dodatkowo, `malloc` ma nieprzewidywalny czas wykonania.
    

#### 4. Krótkie funkcje

Funkcja nie powinna być dłuższa niż to, co mieści się na jednej kartce papieru (ok. 60 linii).

- **Pod maską:** Chodzi o złożoność cykliczną (_Cyclomatic Complexity_). Im mniejsza funkcja, tym łatwiej udowodnić jej poprawność formalną i przetestować wszystkie możliwe ścieżki wykonania.
    

#### 5. Gęstość asercji (Assertions)

Średnio co najmniej 2 asercje (`assert`) na funkcję.

- **Pod maską:** Asercje to weryfikacja niezmienników (_invariants_) w czasie rzeczywistym. W teorii Hoare'a (logika poprawności programów) mamy trójki $\{P\}C\{Q\}$, gdzie $P$ to warunek wstępny (precondition), a $Q$ to warunek końcowy (postcondition). Asercje w kodzie są operacyjną implementacją tej teorii – "łapią" błędy w stanie programu w miejscu ich wystąpienia, a nie w momencie awarii systemu.
    

#### 6. Minimalny zasięg zmiennych (Scope)

Zmienne powinny być deklarowane w najmniejszym możliwym zasięgu.

- **Pod maską:** Ogranicza to dostęp do danych i zapobiega niezamierzonym efektom ubocznym (_side effects_). Ułatwia to analizę przepływu danych (_data-flow analysis_).
    

#### 7. Sprawdzanie wartości zwracanych

Każda funkcja zwracająca wartość musi być sprawdzona (nie wolno ignorować np. błędów `printf` czy operacji we/wy).

- **Pod maską:** W C ignorowanie błędów jest domyślne. Wymuszenie sprawdzenia (nawet przez rzutowanie na `(void)`) zmusza programistę do jawnego podjęcia decyzji o obsłudze błędu.
    

#### 8. Ograniczenie preprocesora

Użycie `#define`, `#ifdef` itp. ograniczone tylko do dołączania plików nagłówkowych i prostych stałych. Zakaz skomplikowanych makr rozwijających kod.

- **Pod maską:** Preprocesor działa przed kompilatorem. Makra są trudne do analizy przez narzędzia statyczne, ponieważ "zaciemniają" kod. Analizator widzi wynik rozwinięcia, a programista widzi makro – to rodzi dysonans poznawczy i błędy.
    

#### 9. Ograniczenie wskaźników

Zabronione jest używanie więcej niż jednego poziomu dereferencji (np. `**p` jest zakazane). Wskaźniki funkcyjne są również mocno ograniczone.

- **Pod maską:** Wskaźniki powodują tzw. _pointer aliasing_ (sytuację, gdzie dwa różne wskaźniki mogą odwoływać się do tego samego miejsca w pamięci). To koszmar dla optymalizatorów i analizatorów statycznych. Ograniczenie dereferencji upraszcza model pamięci.
    

#### 10. Kompilacja bez ostrzeżeń

Kod musi się kompilować z najbardziej restrykcyjnymi ustawieniami kompilatora (np. `-Wall -Wpedantic` w GCC) bez żadnego ostrzeżenia.

- **Pod maską:** Ostrzeżenia kompilatora to często błędy logiczne lub niezdefiniowane zachowania (_Undefined Behavior_), które standard C dopuszcza. Ich eliminacja usuwa "szarą strefę" języka.
    

---

### Gdzie mogę to wykorzystać?

Choć standard powstał w NASA, jego zasady są uniwersalne dla każdego, kto chce pisać solidny kod w C/C++:

1. **Systemy Embedded:** Programowanie mikrokontrolerów (STM32, AVR, ESP), sterowniki PLC.
    
2. **Automotive:** Standard MISRA C (używany w motoryzacji) pokrywa się w dużej mierze z Power of Ten.
    
3. **High-Frequency Trading (HFT):** Tam, gdzie szybkość i niezawodność są kluczowe, a GC (Garbage Collector) jest wrogiem.
    
4. **Projekty studenckie:** Zastosowanie tych zasad (np. brak malloc, brak rekurencji) w projekcie na zaliczenie (np. system operacyjny, prosta gra w C) pokaże prowadzącemu Twoją dojrzałość inżynierską.