---
description: Migruje CLAUDE.md i .claude/commands do formatu GitHub Copilot (copilot-instructions.md + prompt files)
---

Zrób migrację konfiguracji Claude Code do formatu GitHub Copilot w tym projekcie (IntelliJ, jedno okno, kilka serwisów jako moduły). Nie usuwaj żadnych istniejących plików — tylko dodaj nowe.

1. Znajdź plik CLAUDE.md w korzeniu projektu. Skopiuj jego treść BEZ ŻADNYCH ZMIAN do nowego pliku .github/copilot-instructions.md (utwórz folder .github, jeśli nie istnieje).

2. Znajdź folder .claude/commands/ w korzeniu projektu. Dla KAŻDEGO pliku *.md w tym folderze (pomiń README.md, jeśli tam jest) utwórz odpowiadający mu plik w .github/prompts/<ta-sama-nazwa>.prompt.md, według tych zasad:
   - Odczytaj z frontmattera (blok między --- na górze pliku) tylko pole "description".
   - Nowy frontmatter ma zawierać WYŁĄCZNIE:
     ---
     description: <ta sama wartość>
     ---
   - Pomiń pola "argument-hint" i "allowed-tools" z frontmattera — zamiast tego, zaraz pod frontmatterem dodaj linię:
     > Wywołanie: /<nazwa-pliku-bez-rozszerzenia> <argument> — argument wpisz w tej samej linijce w czacie.
   - W treści właściwej (poniżej frontmattera) zamień KAŻDE wystąpienie tekstu "$ARGUMENTS" na tekst: "(wartość podana po komendzie w czacie)". Poza tą jedną zamianą nie zmieniaj ani nie skracaj reszty treści — ma zostać identyczna co do słowa.

3. Po skończeniu pokaż mi listę utworzonych plików (ścieżka -> ścieżka).

Nie kasuj CLAUDE.md ani .claude/commands/ — mają zostać nietknięte, korzystam z nich też w Claude Code.
