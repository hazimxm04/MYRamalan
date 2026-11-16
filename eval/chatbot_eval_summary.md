# Chatbot evaluation

| set-up | questions | passed | pass rate |
|---|---|---|---|
| router | 27 | 23 | 85% |
| agent | 27 | 27 | 100% |

## Why answers failed

| set-up | cause | count |
|---|---|---|
| router | missing_content | 4 |

## By question type (passed / asked)

| type | router | agent |
|---|---|---|
| flips | 2/2 | 2/2 |
| forecast | 4/4 | 4/4 |
| guardrail | 1/5 | 5/5 |
| method | 2/2 | 2/2 |
| multi_step | 1/1 | 1/1 |
| seat | 1/1 | 1/1 |
| sentiment | 1/1 | 1/1 |
| trap_ambiguous | 1/1 | 1/1 |
| trap_confidence | 1/1 | 1/1 |
| trap_out_of_scope | 1/1 | 1/1 |
| trap_unavailable | 2/2 | 2/2 |
| validation | 4/4 | 4/4 |
| what_if | 2/2 | 2/2 |

Pass = the right words appear, the expected tools were called, and (for tool-based set-ups) every number in the answer is supported by a tool result. Wording checks are crude; read the failed answers in `eval/chatbot_eval_results.csv`.
