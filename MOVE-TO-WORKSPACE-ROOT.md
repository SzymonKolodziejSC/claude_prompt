Musisz przenieść pliki konfiguracji GitHub Copilot z podfolderu do korzenia bieżącego workspace'u.

1. Znajdź w bieżącym workspace podfolder zawierający ścieżkę .github/prompts z plikami *.prompt.md (to sklonowane repo o nazwie zaczynającej się od "claude_prompt" lub podobnej — zawiera .github/prompts/*.prompt.md, .github/copilot-instructions.md, global-copilot-instructions.md, README-COPILOT-JETBRAINS.md).

2. Przenieś z tego podfolderu do korzenia workspace'u (na ten sam poziom co foldery mikroserwisów typu 29126-*):
   - cały folder .github (razem z podfolderem prompts w środku) -> <root>/.github
   - plik global-copilot-instructions.md -> <root>/global-copilot-instructions.md
   - plik README-COPILOT-JETBRAINS.md -> <root>/README-COPILOT-JETBRAINS.md

   Jeśli w korzeniu workspace już istnieje folder .github (np. z wcześniejszej, częściowej próby) — scal zawartość, nie nadpisuj na ślepo, zapytaj mnie jeśli są konflikty nazw plików.

3. Po przeniesieniu, USUŃ CAŁKOWICIE oryginalny sklonowany podfolder repozytorium (razem z jego ukrytym folderem .git w środku) — to był tylko nośnik do przeniesienia plików, treść już jest bezpiecznie na GitHubie w tym repo, więc nie musi zostawać jako osobne zagnieżdżone repo git wewnątrz tego workspace'u.

4. Pokaż mi ostateczną listę plików w <root>/.github/ (rekurencyjnie) oraz potwierdź, że podfolder ze sklonowanym repo został usunięty.

Nie ruszaj żadnych folderów mikroserwisów (29126-*), .idea, ani żadnych innych plików projektu.
