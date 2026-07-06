# Git

> **Poziom:** 🟢 podstawowy (użycie) → 🔴 (model, sytuacje awaryjne)
> **Wymagana wiedza:** brak

Git to system kontroli wersji — codzienne narzędzie każdego developera i częsty temat rozmów (zwłaszcza „jak byś wyszedł z tej sytuacji"). Wartość nie w recytowaniu komend, lecz w **modelu mentalnym**: gdy rozumiesz, czym są commity, gałęzie i wskaźniki, komendy stają się oczywiste, a katastrofy odwracalne.

## Model mentalny (fundament)

- **Commit** — niemutowalna migawka całego drzewa plików + metadane + wskaźnik na rodzica; identyfikowany hashem. Historia to **graf skierowany** commitów.
- **Branch** — po prostu **ruchomy wskaźnik** na commit (tani, to jeden plik z hashem). Tworzenie gałęzi = utworzenie wskaźnika, nie kopiowanie plików.
- **HEAD** — wskaźnik na „gdzie jesteś" (zwykle na branch).
- **Trzy obszary:** working directory (pliki) → **staging (index)** (`git add`) → repozytorium (`git commit`). Staging to poczekalnia: wybierasz, co wejdzie do commita.

Zrozumienie „branch = wskaźnik" wyjaśnia, czemu rebase/reset/merge są tanie i czemu „stracone" commity zwykle da się odzyskać (reflog).

## Codzienny workflow

```bash
git status / git diff                 # co się zmieniło (unstaged) / git diff --staged
git add -p                            # interaktywnie wybierz fragmenty (czyste commity!)
git commit -m "feat: dodaj koszyk"
git switch -c feature/cart            # nowa gałąź (nowsze niż checkout -b)
git switch main
git pull --rebase                     # aktualizuj bez merge-commita
git push -u origin feature/cart
```

- `git add -p` (patch) — commituj logiczne całości, nie „wszystko naraz"; podstawa czytelnej historii.
- `git switch`/`git restore` — nowsze, jednoznaczne komendy (dawniej przeciążony `checkout`).

## Merge vs rebase — klasyka rozmów

Oba integrują zmiany z jednej gałęzi do drugiej, inaczej kształtując historię:

- **Merge** — tworzy **merge commit** łączący dwie linie; zachowuje faktyczny przebieg (historia „prawdziwa", ale z rozgałęzieniami).
- **Rebase** — **przenosi** Twoje commity na czubek innej gałęzi, tworząc ich **nowe kopie** (nowe hashe); historia liniowa, czytelna.

**Złota zasada rebase:** nie rebase'uj gałęzi **współdzielonej/opublikowanej** (przepisujesz historię, którą inni już mają → rozjazd). Typowy workflow: rebase lokalnej gałęzi feature na `main` (sprzątanie przed PR), merge do `main` (zachowanie kontekstu). `git pull --rebase` unika śmieciowych merge-commitów przy synchronizacji.

- **Squash** (przy merge PR) — zbija feature w jeden commit: czysta historia `main`, ale utrata granularności.

## Cofanie i naprawa (najważniejsze na rozmowie)

```bash
git commit --amend                    # popraw OSTATNI commit (wiadomość/zawartość) — tylko lokalnie!
git restore --staged plik             # usuń ze stagingu (unstage)
git restore plik                      # porzuć zmiany w pliku (uwaga: nieodwracalne dla unstaged)
git revert <hash>                     # NOWY commit cofający zmiany — bezpieczne dla wspólnej historii
git reset --soft HEAD~1               # cofnij commit, zmiany zostają w stagingu
git reset --mixed HEAD~1              # cofnij commit + unstage (domyślne), zmiany w working dir
git reset --hard HEAD~1               # cofnij commit i WYRZUĆ zmiany — niebezpieczne
git reflog                            # historia RUCHÓW HEAD — koło ratunkowe (odzysk "straconych" commitów)
git stash / git stash pop             # schowaj zmiany na bok i przywróć
```

**Kluczowe rozróżnienie:** `revert` (bezpieczny — dodaje commit) vs `reset` (przepisuje — tylko dla niepushniętej historii). „Zniszczyłem historię" ratuje zwykle **`git reflog`** — HEAD pamięta poprzednie pozycje, `git reset --hard <hash z reflog>` cofa katastrofę.

## Konflikty

Powstają, gdy dwie gałęzie zmieniły **te same linie**. Git wstawia markery `<<<<<<< ======= >>>>>>>`; rozwiązujesz ręcznie (wybierasz/łączysz), `git add`, kontynuujesz (`git rebase --continue`/`git merge --continue`). Minimalizacja: małe PR-y, częsta synchronizacja z `main`, jasny podział własności plików. `git rerere` zapamiętuje rozwiązania powtarzalnych konfliktów.

## Śledztwo i strategie gałęzi

- `git log --oneline --graph --all`, `git blame plik` (kto/kiedy zmienił linię), `git bisect` (**binarne** szukanie commita, który wprowadził bug — half-splitting po historii; bezcenne).
- **Strategie gałęzi:** *trunk-based* (krótkożyjące gałęzie, częsty merge do `main`, feature flags — sprzyja [CI/CD](../16-devops-dla-frontendu/01-ci-cd.md)) vs *Git Flow* (rozbudowany, gałęzie release/develop — cięższy, dla wydań wersjonowanych). Współczesny frontend skłania się ku trunk-based.
- **Conventional commits** (`feat:`, `fix:`) — [jakość kodu](./04-jakosc-kodu.md); automatyzują changelog/wersjonowanie.

## Pułapki i częste błędy

- `git reset --hard` / `git push --force` na współdzielonej gałęzi — utrata cudzej pracy; używaj `--force-with-lease` (sprawdza, czy ktoś nie dopushował).
- `commit --amend`/rebase gałęzi już opublikowanej — rozjazd historii dla współpracowników.
- Wielkie „god commits" (`git add .` wszystkiego) — nieczytelna historia, trudny review i revert.
- Commitowanie sekretów/`node_modules`/plików build — `.gitignore` + skanery sekretów; sekret w historii = przepisanie historii + rotacja klucza.
- Panika po „utracie" commitów zamiast `git reflog`.
- Długożyjące gałęzie feature — piekło konfliktów przy merge.

## Pytania rekrutacyjne

1. **Merge vs rebase — różnice i kiedy które?** — merge commit vs liniowa historia; złota zasada (nie rebase'uj wspólnej).
2. **`git revert` vs `git reset`?** — bezpieczne cofnięcie nowym commitem vs przepisanie historii.
3. **Zrobiłeś `reset --hard`, straciłeś commity — co robisz?** — `git reflog` + reset na odzyskany hash.
4. **Jak utrzymujesz czytelną historię?** — małe commity (`add -p`), squash/rebase feature, conventional commits.
5. **Jak znajdziesz commit, który wprowadził buga?** — `git bisect`.
6. **Czym jest staging area?** — poczekalnia między working dir a repo; selektywne commity.

## Dalsza lektura

- [Pro Git (darmowa książka)](https://git-scm.com/book) — rozdziały 2, 3, 7
- [Learn Git Branching (interaktywnie)](https://learngitbranching.js.org/)
- [Oh Shit, Git!?!](https://ohshitgit.com/) — wychodzenie z opresji
