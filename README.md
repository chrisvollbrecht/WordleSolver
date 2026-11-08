# Wordle Solver Prototype

Open `index.html` in a browser to try the UI. The app has no build step or package dependencies. If your browser blocks local loading of the word-list files, serve this folder with any static HTTP server.

## Word Data

The bundled lists are a 2022 snapshot from the [3Blue1Brown Wordle data directory](https://github.com/3b1b/videos/tree/master/_2022/wordle/data):

- `allowed-words.txt`: 12,953 accepted guesses.
- `possible-answers.txt`: 2,309 possible answers from the same snapshot.

The answer list and accepted-guess list are separate on purpose. The app validates guesses against the larger list and filters candidates against the answer list. These lists are not a live feed from the NYT and may not reflect later changes.

## Ranking

The app simulates each feedback-consistent accepted guess against the remaining answers, groups answers by the resulting tile pattern, and ranks guesses by the expected number of answers left. Worst-case group size breaks ties; during the first two turns, letter coverage is an additional tie-breaker. Feedback uses Wordle's duplicate-letter rules.

## Past Answers

Past answers are not currently excluded. That requires a separate, date-aware history of actual daily solutions; an undated possible-answer list cannot identify which answers have already been used. A history filter should affect the answer pool only, not the list of words accepted as guesses.

## Strategy Check

I tested four greedy strategies against 250 unique solutions sampled from random 2022 dates using the [NYT dated puzzle endpoint](https://www.nytimes.com/svc/wordle/v2/2022-01-01.json). The sample was selected with seed `20261108`. Each strategy could use any accepted guess, stopped at six guesses, and used the same 2,309-word answer pool.

| Strategy | First guess | Average guesses | Solved within six |
| --- | --- | ---: | ---: |
| Minimize expected answers remaining | `roate` | 3.568 | 250 / 250 |
| Maximize feedback-pattern entropy | `soare` | 3.648 | 250 / 250 |
| Maximize distinct-letter coverage for two turns, then expected remaining | `roate` | 3.776 | 250 / 250 |
| Weight green hits 2x yellow hits for two turns, then expected remaining | `soare` | 3.768 | 250 / 250 |

The expected-remaining policy performed best among these four, by about 0.08 guesses versus entropy and 0.20 versus the early letter-focused heuristics. This is not a proof of the globally minimum average: these are greedy policies, not an exhaustive optimal decision tree. The benchmark allowed any accepted guess, while the UI now restricts recommendations to guesses consistent with prior feedback; rerun the benchmark before treating its numbers as a direct evaluation of the current UI policy. The candidate pool was held fixed for all sampled dates and therefore still included some answers already used by that date; this isolates ranking behavior but is not yet a date-accurate replay with the no-repeat advantage.