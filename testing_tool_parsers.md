# Setup dev machine

```
ssh aws-dev-instance

# install vllm
tmux new-session -A -s vllm
pip install "vllm==0.8.5.post1"

# get the custom llama4 lark tool parser
mkdir -p ~/src
cd ~/src
git clone https://github.com/bbrowning/vllm.git
cd ~/src/vllm
git checkout llama4-lark-tool-parser

# install llama-stack-evals with bfclv3-api fixes
:new -s llama-stack-evals
cd ~/src
git clone https://github.com/bbrowning/llama-stack-evals.git
cd llama-stack-evals
git checkout bfclv3-api-fixes
pyenv shell 3.12
python -m venv venv
source venv/bin/activate
pip install -e .
pip install git+https://github.com/ShishirPatil/gorilla.git@main#subdirectory=berkeley-function-call-leaderboard`
```

# RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic

## bfclv3

run vLLM:

```
mkdir -p ~/tmp
cd ~/tmp

vllm serve RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
            --port 8001 \
            --enable-auto-tool-choice \
            --chat-template ~/src/vllm/examples/tool_chat_template_llama4_pythonic.jinja \
            --tool-parser-plugin ~/src/vllm/vllm/entrypoints/openai/tool_parsers/llama4_pythonic_tool_parser.py \
            --tool-call-parser llama4_pythonic \
            --gpu-memory-utilization 0.99 \
            --enforce-eager \
            --max-model-len 128000 \
            --tensor-parallel-size 4 \
            2>&1 | tee vllm_bfclv3.log
```

run the tests:

```
cd ~/src/llama-stack-evals
source venv/bin/activate

DEBUG_LOG_FILE=bfclv3-debug.log \
ERROR_LOG_FILE=bfclv3-error.log \
MAX_RETRIES=1 \
OPENAI_API_KEY=fake \
llama-stack-evals run-benchmarks \
  --benchmarks bfclv3 \
  --provider vllm \
  --model RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
  --max-parallel-generations 20
```

### results

```
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4441/4441 [1:09:06<00:00,  1.07it/s]
Running 1 graders on 4441 generations
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4441/4441 [00:04<00:00, 905.15it/s]

Saved raw results CSV to /home/ec2-user/.llama/evals/cache/20250527-1307-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3.csv
Saved detailed results JSON to /home/ec2-user/.llama/evals/cache/20250527-1307-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3.json
Saved run summary report to /home/ec2-user/.llama/evals/cache/20250527-1307-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/report.json

                                                    RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic                                                     
┏━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Benchmark ID ┃ Subset                            ┃ Result                                                                                                ┃
┡━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ bfclv3       │ Overall                           │ {"accuracy": 0.6514298581400585, "num_correct": 2893.0, "num_incorrect": 1548.0, "num_total": 4441.0} │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ total_overall_accuracy            │ {"accuracy": 0.544039291179229, "num_total": 4441.0}                                                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ summary_ast_non_live              │ {"accuracy": 0.8091666666666666, "num_total": 1150.0}                                                 │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ simple_ast_non_live               │ {"accuracy": 0.7916666666666666, "num_total": 550.0}                                                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ multiple_ast_non_live             │ {"accuracy": 0.92, "num_correct": 184.0, "num_incorrect": 16.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ parallel_ast_non_live             │ {"accuracy": 0.765, "num_correct": 153.0, "num_incorrect": 47.0, "num_total": 200.0}                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ parallel_multiple_ast_non_live    │ {"accuracy": 0.76, "num_correct": 152.0, "num_incorrect": 48.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ overall_accuracy_live             │ {"accuracy": 0.7347845402043536, "num_total": 2251.0}                                                 │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ python_simple_ast_live            │ {"accuracy": 0.810077519379845, "num_correct": 209.0, "num_incorrect": 49.0, "num_total": 258.0}      │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ python_multiple_ast_live          │ {"accuracy": 0.7236467236467237, "num_correct": 762.0, "num_incorrect": 291.0, "num_total": 1053.0}   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ python_parallel_ast_live          │ {"accuracy": 0.6875, "num_correct": 11.0, "num_incorrect": 5.0, "num_total": 16.0}                    │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ python_parallel_multiple_ast_live │ {"accuracy": 0.6666666666666666, "num_correct": 16.0, "num_incorrect": 8.0, "num_total": 24.0}        │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ overall_accuracy_multi_turn       │ {"accuracy": 0.08, "num_total": 800.0}                                                                │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ multi_turn_base                   │ {"accuracy": 0.09, "num_correct": 18.0, "num_incorrect": 182.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ multi_turn_miss_func              │ {"accuracy": 0.08, "num_correct": 16.0, "num_incorrect": 184.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ multi_turn_miss_param             │ {"accuracy": 0.095, "num_correct": 19.0, "num_incorrect": 181.0, "num_total": 200.0}                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ multi_turn_long_context           │ {"accuracy": 0.055, "num_correct": 11.0, "num_incorrect": 189.0, "num_total": 200.0}                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ total_relevance                   │ {"accuracy": 0.8888888888888888, "num_correct": 16.0, "num_incorrect": 2.0, "num_total": 18.0}        │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3       │ total_irrelevance                 │ {"accuracy": 0.7878117913832199, "num_total": 1122.0}                                                 │
└──────────────┴───────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────┘
Evaluation complete! Cache saved to 
/home/ec2-user/.llama/evals/cache/20250527-1307-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/
```

## bfclv3-api with default llama4_pythonic parser

run vLLM:

```
mkdir -p ~/tmp
cd ~/tmp

vllm serve RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
            --port 8001 \
            --enable-auto-tool-choice \
            --chat-template ~/src/vllm/examples/tool_chat_template_llama4_pythonic.jinja \
            --tool-parser-plugin ~/src/vllm/vllm/entrypoints/openai/tool_parsers/llama4_pythonic_tool_parser.py \
            --tool-call-parser llama4_pythonic \
            --gpu-memory-utilization 0.99 \
            --enforce-eager \
            --max-model-len 128000 \
            --tensor-parallel-size 4 \
            2>&1 | tee vllm_llama4_pythonic.log
```

run the tests:

```
cd ~/src/llama-stack-evals
source venv/bin/activate

DEBUG_LOG_FILE=bfclv3-api-llama4_pythonic-debug.log \
ERROR_LOG_FILE=bfclv3-api-llama4_pythonic-error.log \
MAX_RETRIES=1 \
OPENAI_API_KEY=fake \
llama-stack-evals run-benchmarks \
  --benchmarks bfclv3-api \
  --provider vllm \
  --model RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
  --max-parallel-generations 20
```


## bfclv3-api with custom Lark parser

run vLLM:

```
mkdir -p ~/tmp
cd ~/tmp

vllm serve RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
            --port 8001 \
            --enable-auto-tool-choice \
            --chat-template ~/src/vllm/examples/tool_chat_template_llama4_pythonic.jinja \
            --tool-parser-plugin ~/src/vllm/vllm/entrypoints/openai/tool_parsers/lark_pythonic_tool_parser.py \
            --tool-call-parser lark_pythonic \
            --gpu-memory-utilization 0.99 \
            --enforce-eager \
            --max-model-len 128000 \
            --tensor-parallel-size 4 \
            2>&1 | tee vllm_lark_pythonic.log
```


run the tests:

```
cd ~/src/llama-stack-evals
source venv/bin/activate

DEBUG_LOG_FILE=bfclv3-api-debug.log \
ERROR_LOG_FILE=bfclv3-api-error.log \
MAX_RETRIES=1 \
OPENAI_API_KEY=fake \
llama-stack-evals run-benchmarks \
  --benchmarks bfclv3-api \
  --provider vllm \
  --model RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic \
  --max-parallel-generations 20
```

### results

```
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4441/4441 [1:11:22<00:00,  1.04it/s]
Running 1 graders on 4441 generations
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4441/4441 [00:04<00:00, 917.61it/s]

Saved raw results CSV to /home/ec2-user/.llama/evals/cache/20250527-1428-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-api.csv
Saved detailed results JSON to /home/ec2-user/.llama/evals/cache/20250527-1428-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/bfclv3-api.json
Saved run summary report to /home/ec2-user/.llama/evals/cache/20250527-1428-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/report.json

                                                    RedHatAI/Llama-4-Scout-17B-16E-Instruct-FP8-dynamic                                                     
┏━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Benchmark ID ┃ Subset                            ┃ Result                                                                                                ┃
┡━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ bfclv3-api   │ Overall                           │ {"accuracy": 0.6719207385723936, "num_correct": 2984.0, "num_incorrect": 1457.0, "num_total": 4441.0} │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ total_overall_accuracy            │ {"accuracy": 0.5685928723036674, "num_total": 4441.0}                                                 │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ summary_ast_non_live              │ {"accuracy": 0.873125, "num_total": 1150.0}                                                           │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ simple_ast_non_live               │ {"accuracy": 0.7425, "num_total": 550.0}                                                              │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ multiple_ast_non_live             │ {"accuracy": 0.93, "num_correct": 186.0, "num_incorrect": 14.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ parallel_ast_non_live             │ {"accuracy": 0.92, "num_correct": 184.0, "num_incorrect": 16.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ parallel_multiple_ast_non_live    │ {"accuracy": 0.9, "num_correct": 180.0, "num_incorrect": 20.0, "num_total": 200.0}                    │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ overall_accuracy_live             │ {"accuracy": 0.7481119502443359, "num_total": 2251.0}                                                 │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ python_simple_ast_live            │ {"accuracy": 0.813953488372093, "num_correct": 210.0, "num_incorrect": 48.0, "num_total": 258.0}      │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ python_multiple_ast_live          │ {"accuracy": 0.724596391263058, "num_correct": 763.0, "num_incorrect": 290.0, "num_total": 1053.0}    │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ python_parallel_ast_live          │ {"accuracy": 0.75, "num_correct": 12.0, "num_incorrect": 4.0, "num_total": 16.0}                      │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ python_parallel_multiple_ast_live │ {"accuracy": 0.75, "num_correct": 18.0, "num_incorrect": 6.0, "num_total": 24.0}                      │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ overall_accuracy_multi_turn       │ {"accuracy": 0.0875, "num_total": 800.0}                                                              │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ multi_turn_base                   │ {"accuracy": 0.115, "num_correct": 23.0, "num_incorrect": 177.0, "num_total": 200.0}                  │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ multi_turn_miss_func              │ {"accuracy": 0.015, "num_correct": 3.0, "num_incorrect": 197.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ multi_turn_miss_param             │ {"accuracy": 0.11, "num_correct": 22.0, "num_incorrect": 178.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ multi_turn_long_context           │ {"accuracy": 0.11, "num_correct": 22.0, "num_incorrect": 178.0, "num_total": 200.0}                   │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ total_relevance                   │ {"accuracy": 0.7777777777777778, "num_correct": 14.0, "num_incorrect": 4.0, "num_total": 18.0}        │
├──────────────┼───────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ bfclv3-api   │ total_irrelevance                 │ {"accuracy": 0.8072845804988662, "num_total": 1122.0}                                                 │
└──────────────┴───────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────┘
Evaluation complete! Cache saved to 
/home/ec2-user/.llama/evals/cache/20250527-1428-RedHatAI-Llama-4-Scout-17B-16E-Instruct-FP8-dynamic/ 
```
