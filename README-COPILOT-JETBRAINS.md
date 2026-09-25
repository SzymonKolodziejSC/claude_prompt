# Claude Code → GitHub Copilot w IntelliJ (plugin JetBrains)

Ważna korekta względem poprzedniej wersji tej paczki: **JetBrains obsługuje inny, węższy zestaw plików niż VS Code.**

| | VS Code | JetBrains (IntelliJ) |
|---|---|---|
| Instrukcje repo | `.github/copilot-instructions.md`, `AGENTS.md`, **`CLAUDE.md`** | **tylko** `.github/copilot-instructions.md` (jeden plik) |
| Instrukcje globalne | user-level w ustawieniach | `global-copilot-instructions.md` w osobnym folderze systemowym |
| Instrukcje per-typ-pliku (`*.instructions.md` z `applyTo`) | tak | **nie** |
| Prompt files (`/nazwa`) | tak, z bogatym frontmatterem (`agent`, `tools`, `${input:...}`) | tak, ale prościej — bez potwierdzonego wsparcia dla tych pól frontmatter |

Czyli: **`CLAUDE.md` w IntelliJ nie jest rozpoznawany.** Trzeba mieć osobno `.github/copilot-instructions.md`.

## 1. Instrukcje repo — `.github/copilot-instructions.md`

W paczce jest gotowy plik (treść Twojego `CLAUDE.md`, skopiowana bez zmian). Włóż go do repo, które akurat otwierasz w IntelliJ jako projekt.

**To zależy od tego, jak trzymasz serwisy w IntelliJ:**
- **Jeden projekt IntelliJ na cały `~/IdeaProjects`** (multi-module: `service-a`, `service-b`... jako attached moduły/content roots w jednym `.idea`) → wystarczy **jeden** `.github/copilot-instructions.md` w korzeniu tego projektu (czyli w `~/IdeaProjects/.github/`). Copilot powinien go zobaczyć dla całego otwartego projektu.
- **Każdy serwis to osobny projekt/okno IntelliJ** → plik musi być **w każdym** `service-*/.github/copilot-instructions.md` osobno (skopiuj/synchronizuj tę samą treść do każdego repo — JetBrains nie ma tu odpowiednika multi-root workspace z VS Code).

Sprawdzenie działania: w oknie Copilot Chat spójrz na listę **References** pod odpowiedzią — powinien pojawić się `copilot-instructions.md`.

## 2. Odpowiednik `~/.claude/CLAUDE.md` (globalne, dla wszystkich projektów)

Plik `global-copilot-instructions.md` (w paczce, do skopiowania) w:
- **macOS / Linux:** `~/.config/github-copilot/intellij/global-copilot-instructions.md`
- **Windows:** `%LOCALAPPDATA%\github-copilot\intellij\global-copilot-instructions.md`

Albo wygodniej: **Settings → Tools → GitHub Copilot → Customizations** → sekcja "Copilot Instructions" → **Global** (edytujesz z poziomu IDE, bez szukania folderu).

Ten sam ekran, opcja **Workspace**, to skrót do edycji punktu 1. powyżej.

## 3. Slash-komendy → Prompt files

`.github/prompts/*.prompt.md` działają też w JetBrains (funkcja w publicznym preview). W paczce masz uproszczone wersje Twoich 8 komend — bez pól `agent:`/`tools:`/`${input:...}`, bo dokumentacja JetBrains ich nie opisuje (prawdopodobnie jeszcze nieobsługiwane) — żeby nic nie wyświetlało się jako martwy tekst w promptcie.

Jak wywołać: w oknie czatu wpisujesz `/` i nazwę pliku, np. `/start-task`, i **od razu w tej samej linijce dopisujesz argument**, np.:

```
/start-task ZFD-12345
```

Komenda ma i tak wbudowany fallback „jeśli puste, zapytaj", więc jak zapomnisz dopisać ticket, model zapyta sam.

Możesz też tworzyć/edytować prompty z poziomu UI: **Settings → Tools → GitHub Copilot → Edit Settings → Customizations → Prompt Files → Workspace**.

## 3a. Alternatywa: niech Copilot sam sobie to zrobi

W paczce jest dodatkowo `.github/prompts/migrate-claude-to-copilot.prompt.md`. Wystarczy wrzucić folder `.github/prompts/` do korzenia projektu, otworzyć Copilot Chat w **trybie agent** i wpisać:

```
/migrate-claude-to-copilot
```

Sam znajdzie Twój `CLAUDE.md` i `.claude/commands/`, i rozłoży je jako `.github/copilot-instructions.md` + resztę `.github/prompts/*.prompt.md` — bez ręcznego kopiowania treści. Traktuj to jako alternatywę dla ręcznego przenoszenia plików z tej paczki (nie rób obu na raz, żeby nie było duplikatów).

## 4. Narzędzia (`allowed-tools` z Claude Code)

IntelliJ-owy plugin Copilota na razie nie ma udokumentowanego, granularnego dobierania narzędzi per-prompt tak jak VS Code (ikonka "Configure tools"). W praktyce agent w trybie czatu ma dostęp do standardowego zestawu akcji edytora/terminala IntelliJ — zostawiłem oryginalną listę z Claude Code jako komentarz w treści każdego prompta, żebyś wiedział, czego dana komenda oczekiwała (np. `mcp__atlassian__*` = potrzebny podłączony serwer MCP Atlassian, o ile IntelliJ Copilot u Ciebie wspiera MCP — warto sprawdzić w Settings → GitHub Copilot → MCP).
