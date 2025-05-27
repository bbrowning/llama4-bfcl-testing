# Llama 4 tool call testing with Berkeley Function Calling Leaderboard

This repository has results comparing Llama 4 results on the Berkeley Function Calling Leaderboard tests as run via the meta-llama/llama-stack-evals project. We run with its `bfclv3` and `bfclv3-api` configurations, which test doing client-side prompting and parsing compared to server-side chat templates and tool call parsers.

The test setup is using vLLM 0.8.5.post1, sometimes with a custom tool call parser.


## bfclv3 results

See [results/RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-20250527-1307](results/RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-20250527-1307)

## bfclv3-api results with llama4_pythonic tool parser (vLLM default)

testing in progress...

## bfclv3-api results with a custom Lark-grammar and tool parser

See [results/RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-api-lark_pythonic-20250527-1428](results/RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-api-lark_pythonic-20250527-1428)
