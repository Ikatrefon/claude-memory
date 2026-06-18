# Zasady startowe projektu — kontrakt anty-barokowy

> Ten plik jest wczytywany na początku projektu. Jest kontraktem, nie sugestią.
> Stosujesz go domyślnie, bez przypominania. Jeśli odstępujesz od zasady — nazywasz to wprost i podajesz powód.

---

## 0. Zasada fundamentalna

Celem nie jest kod „solidny" ani „kompletny". Celem jest **najmniejszy kod, który niesie wymaganą funkcję**.

Baroku nie da się wyeliminować — to twój domyślny tryb przy niedospecyfikowaniu, i wraca w trakcie iteracji nawet po czystym starcie. Zadaniem tych zasad jest go **ograniczać** i **dawać przyzwolenie na kasowanie**. Usuwanie działającego kodu jest progresem, nie regresem.

Gdy zasada „chudości" jest w konflikcie z zasadą „solidności" — chudość wygrywa, dopóki nie ma dowodu (test, realny przypadek), że solidność jest potrzebna.

---

## 1. Źródło baroku — żebyś wiedział, czego unikać

Twój przerost to nie ambicja inżynierska. To **spekulatywna ogólność**: budujesz pod wyobrażone przyszłe potrzeby, bo na takim kodzie byłeś trenowany. Konkretne objawy, których masz unikać:

- abstrakcja użyta w jednym miejscu (interfejs z jedną implementacją, wrapper przekazujący wywołanie dalej);
- warstwa pośrednia bez powodu poza „tak się robi";
- obsługa błędów dla stanów niemożliwych przy danych typach;
- config dla wartości, która ma jeden stan;
- komentarz opisujący to, co kod już mówi.

Każdy z powyższych jest domyślnie **zakazany**, nie „odradzany".

---

## 2. Zanim napiszesz pierwszą linię

**2.1. Non-goals przed goals.**
Zanim zaproponujesz cokolwiek, wypisz explicite, czego projekt **nie** robi i czego **nie** obsługuje. Każdy niewypełniony non-goal to warstwa, której nie zbudujesz. Jeśli nie dostałeś non-goals — zapytaj o nie, zanim zaczniesz.

**2.2. Walking skeleton najpierw.**
Najmniejsza rzecz działająca end-to-end przed jakąkolwiek rozbudową. Żadnego rusztowania, zanim wiadomo, co na nim stanie. Pierwszy build ma być celowo chudy, nie kompletny.

**2.3. Reguła trzech (YAGNI).**
Żadnej abstrakcji, dopóki ten sam wzorzec nie wystąpi **trzeci** raz. Dwie kopie zostają dwiema kopiami. Abstrahujesz dopiero, gdy duplikacja jest udowodniona, nie przewidziana.

---

## 3. W trakcie generowania

- **Read before write.** Jeśli istnieje kod projektu — przeczytaj go i reużyj. Nie wymyślaj na nowo. Reinwencja jest głównym źródłem przyrostu w projekcie wieloplikowym.
- **Prymitywy przed zależnością.** Sięgnij po bibliotekę standardową / wbudowane w framework, zanim dołożysz pakiet. Każdy import to trwała powierzchnia i dług. Dodanie zależności wymaga uzasadnienia jednym zdaniem.
- **Bez obsługi niemożliwego.** Nie pisz gałęzi dla stanów wykluczonych przez typy. Zaufaj typom.
- **Komentarz tylko nieoczywisty.** Komentuj *dlaczego*, nigdy *co*. Jeśli komentarz opisuje działanie kodu — usuń go i popraw nazwę.
- **Bez przedwczesnej parametryzacji.** Hardcode jednej wartości jest lepszy niż config dla jednego stanu.

---

## 4. Jak zlecać cięcie (trim-pass)

„Zoptymalizuj" i „bądź oszczędny" są zakazane jako instrukcje — są puste, model na nie dopisuje. Cięcie zlecaj operacjami jednoznacznymi i mierzalnymi:

- „zinline'uj każdą funkcję użytą w jednym miejscu";
- „usuń każdą abstrakcję z jedną implementacją";
- „cel: −X% LOC bez zmiany zachowania, pokaż diff".

**Rola recenzenta, nie autora.** Trim-pass rób w osobnej sesji / osobnym kontekście, gdzie występujesz jako recenzent tnący kod do celu liczbowego — nie jako autor broniący własnego. Autor i krytyk w jednym kontekście tną gorzej.

**Test jako warunek cięcia.** Nie tnij kodu bez testu zachowania, który zweryfikuje, że cięcie niczego nie zepsuło. Testuj zachowanie, nie implementację — inaczej testy same stają się barokiem.

---

## 5. Mierzalność — egzekucja przez strukturę, nie przez czujność

Barok ma narastać widocznie, nie po cichu. Tooling wpinasz **od dnia pierwszego**, na czystym projekcie — zapadka działa tylko od zera.

Barok mierzalny rozkłada się na dwie osie:

- **Martwy / nieużywany kod** — eksporty, pliki, zależności, do których nic nie prowadzi. To barok akumulacyjny (gruz po iteracjach).
- **Złożoność** — kod używany, ale przerośnięty wewnętrznie (zagnieżdżenia, defensywne gałęzie). To barok splątany.

### Dla projektów TS / JS (Next.js itd.)

`package.json`:
```json
"scripts": {
  "knip": "knip",
  "knip:ci": "knip --no-progress"
}
```

- **knip** — martwy kod, nieużywane eksporty/pliki/zależności, duplikaty eksportów. Rozumie konwencje Next.js (`page.tsx`, `layout.tsx`, route handlery) przez pluginy, więc nie flaguje ich jako nieużywanych. Używaj knip, **nie** ts-prune (ts-prune jest w trybie maintenance i błędnie oznacza `export default` w Next.js).
- **eslint** — złożoność: reguły `complexity`, `max-depth`, `max-lines-per-function`, `max-params`. Progi startowe: złożoność 10–15, głębokość 3–4, parametry 3–4. To sygnały do tuningu, nie wyrocznie.
- Wpięcie w CI: lokalnie `warn`, w CI `error` (build pada na regresji). To jedyny sposób, by gruz nie wracał bez Twojej wiedzy.
- Uwaga: dynamiczne `import(zmienna)` nie da się przeanalizować statycznie — dodaj takie moduły jako entry pointy w configu knip, inaczej zgłosi fałszywe trafienia. Weryfikuj raport przed kasowaniem.

---

## 6. Czego tooling NIE łapie — granica mierzalności

Static analysis jest **ślepa na spekulatywną ogólność**. Interfejs z jedną implementacją, wrapper przekazujący wywołanie, zbędna warstwa, przez którą wszystko przechodzi — wszystko to jest *technicznie używane* i ma *niską złożoność*, więc przechodzi przez knip i eslint czysto.

Wniosek: tooling (sekcja 5) łapie barok obiektywny — tani, automatyczny ~60%. Barok architektoniczny — abstrakcje-na-zapas, które żyją — wymaga osądu i trim-passu (sekcja 4). To dwie różne warstwy, nie redundancja.

**Pułapka:** czyszczenie martwego kodu daje satysfakcję i bywa zamiennikiem trudniejszej decyzji o nadmiarowej architekturze. Jeśli robisz tylko sekcję 5, dostajesz poczucie porządku przy niezmienionym, przerośniętym fundamencie.

---

## 7. Definition of done

Funkcja jest skończona, gdy:

- [ ] non-goals są spisane i nie zostały po cichu przekroczone;
- [ ] żadna abstrakcja nie ma jednej implementacji (albo jest uzasadniona explicite);
- [ ] żadna zależność nie jest dodana bez uzasadnienia jednym zdaniem;
- [ ] knip przechodzi bez nowych trafień;
- [ ] złożoność mieści się w progach (lub odstępstwo jest świadome i nazwane);
- [ ] przeszedł trim-pass w roli recenzenta;
- [ ] testy zachowania przechodzą.

---

## 8. Dryf

Te zasady nie eliminują baroku — korygują go. Dryf wraca, zwłaszcza w trzeciej, piątej, ósmej iteracji, gdy uwaga jest na nowej funkcji. Wykrywaj go i nazywaj zamiast udawać, że nie istnieje. Sekcje 2.3 i 5 są ważniejsze niż reszta, bo przenoszą egzekucję z pamięci człowieka na zapis i tooling — wszystko, co zależy od „pamiętania, żeby ciąć", zawiedzie.
