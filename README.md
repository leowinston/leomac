# leomac

A mental-math trainer that tracks progress across every device and feeds an AI coaching loop.

Built by Leo Winston. A zetamac-style arithmetic drill with the theming and customization of Monkeytype, a Supabase backend for cross-device history, and per-problem timing that surfaces which problem types are actually slow.

## Why this exists

I practice mental math for quant-finanace interview prep. [Zetamac](https://arithmetic.zetamac.com) has the right drill but keeps no history; [Monkeytype](https://monkeytype.com) has the customization and progress tracking but is a typing test. leomac applies the Monkeytype model to arithmetic: the same fast drill, plus themes, alternative numeral systems, three keypad modes, and a record of every run. Because runs persist to Supabase, an AI assistant with database access reads the history and gives targeted feedback — for example, which operation is slowest by average solve time.

## Features

- Zetamac-faithful game loop
- Configurable ranges per operation (addition, subtraction, multiplication, division) and duration (30 / 60 / 120 / 300 s)
- Monkeytype theme system through CSS custom properties
- Alternative numeral display (Western, Eastern Arabic, Devanagari, Bangla) because I like eastern arabic and other cool alternative number systems
- Three input modes: native keyboard, centered on-screen pad, one-hand pad with a side swap.
- Cross-device sync. Each run stores score, first-try accuracy, average solve time, the run config, and per-problem timing.

## Tech

Single-file front end (HTML, CSS, JavaScript). Supabase (Postgres) back end. Hosted on GitHub Pages. No build step and no framework — the whole client is `index.html`.

## Architecture

```
index.html  ──writes──▶  Supabase (Postgres)
 browser/PWA             public.runs, row-level security
     ▲                        │
     └────reads──── AI assistant (SQL over run history)
```

Light-weight for personal use to track my progress. The client talks to Supabase directly with the JavaScript SDK. Row-level security limits every read and write to the signed-in user. The publishable key is in the client on purpose

## Data model

Table `public.runs`:

| column | type | note |
|---|---|---|
| `id` | uuid | primary key |
| `user_id` | uuid | set to the signed-in user; foreign key to `auth.users` |
| `created_at` | timestamptz | run time |
| `duration_s` | int | session length in seconds |
| `score` | int | problems solved |
| `first_try_accuracy` | numeric | percent solved without a miss |
| `avg_ms_per_problem` | numeric | mean solve time |
| `config` | jsonb | operations, ranges, numeral system, keypad |
| `problems` | jsonb | array of every problem: text, answer, solve time, miss flag |

The `problems` array is the key design choice. It makes per-problem analysis possible, instead of storing only a final score. Tracks:
1. Slowest operation
2. Weakest operand ranges 
