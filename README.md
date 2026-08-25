# librarian

> Cztery skille agentowe, ktore robia z Twojego folderu markdownu prawdziwa baze wiedzy. Agent zapisuje watki, kompiluje je w wiki z backlinkami, odpowiada na pytania **tylko** z tego korpusu i co tydzien sprzata balagan.

Inspiracja: Andrej Karpathy i jego "LLM-as-librarian" - model nie ma pamietac, ma umiec czytac i porzadkowac. Vault to zwykle pliki `.md` (Obsidian, VS Code, cokolwiek), wiec nic Cie nie zamyka w cudzym SaaS.

## Install

```bash
npx skills add heyneuron/librarian
```

Skille trafiaja do `~/.claude/skills/` oraz `~/.agents/skills/` i sa wykrywane przez Claude Code, OpenCode i Gemini CLI. Codex CLI zaciaga je z `~/.agents/skills/`.

Tylko wybrany skill:

```bash
npx skills add heyneuron/librarian -g -a claude-code --skill kb-dump
```

## Setup vaulta

```bash
mkdir -p ~/kb/{wiki,raw,threads,reports}
echo "# index" > ~/kb/wiki/index.md
```

Inna lokalizacja? Ustaw `KB_PATH`:

```bash
echo 'export KB_PATH="$HOME/Documents/kb"' >> ~/.zshrc
```

Vault moze (ale nie musi) byc repo gita. Jak ma remote, skille same commituja i pushuja po kazdym zapisie - dzieki temu kilka osob i kilka maszyn pracuje na jednej bazie.

## Skille

| Skill | Co robi |
|---|---|
| [`kb-dump`](skills/kb-dump/SKILL.md) | Zapisuje aktualny watek rozmowy do `threads/` jako markdown. Surowe materialy (transkrypty, logi) ladu w `raw/`. On-demand - nic nie leci samo w tle. |
| [`kb-compile`](skills/kb-compile/SKILL.md) | Czyta `raw/` i `threads/`, wyciaga koncepty, tworzy/aktualizuje `wiki/` z backlinkami. Tryby: full, scoped, single-source. |
| [`kb-ask`](skills/kb-ask/SKILL.md) | Q&A nad wiki - odpowiedz wylacznie z korpusu, z cytatami zrodel. Raport laduje w `reports/`. |
| [`kb-lint`](skills/kb-lint/SKILL.md) | Health check: sprzecznosci, duplikaty, sieroty, brakujace koncepty, stale daty, index drift. Safe fixy aplikuje sam. |

## Workflow

```
you:   (dluga rozmowa o architekturze)
agent: Warto to zapisac do kb?
you:   dawaj
agent: (kb-dump -> threads/2026-04-13-architektura-kolejki.md,
        potem automatycznie kb-compile -> wiki/message-queue.md
        + backlinki + wpis w index)

you:   drop artykulu do ~/kb/raw/ ; "skompiluj kb"
agent: (kb-compile: 1 nowy concept, 2 update)

you:   co wiemy w kb o kolejkach?
agent: (kb-ask -> reports/2026-04-20-kolejki.md + TLDR w chacie,
        z linkami do zrodel i lista luk w wiki)

you:   (piatek) zlintuj kb
agent: (kb-lint -> raport + poprawiony index)
```

Dwie zasady, na ktorych stoi caly zestaw:

1. **Zero halucynacji** - `kb-ask` odpowiada tylko z vaulta. Jak czegos nie ma, mowi ze nie ma.
2. **Zawsze plik** - odpowiedz w czacie znika, plik zostaje i kumuluje sie z reszta.

## Wymagania

- Agent obslugujacy format [vercel-labs/skills](https://github.com/vercel-labs/skills) (Claude Code, Codex, OpenCode, Gemini CLI)
- `git` - opcjonalnie, tylko jesli chcesz synchronizowac vault miedzy maszynami

## Uwagi

Tresc skilli jest po polsku (bo po polsku powstala), frontmattery po angielsku. Agent i tak pisze notatki w jezyku, w ktorym z nim gadasz.

## Licencja

MIT - [HeyNeuron](https://heyneuron.com)
