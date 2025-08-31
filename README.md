# leakAgent: RL-based Red-teaming Agent for LLM Privacy Leakage

## How to run

### 1. Install the requirements

```bash
pip install -r requirements.txt
```

### 2. train the model

```shell
# launch vllm for target model
vllm serve meta-llama/Llama-3.1-8B-Instruct --tensor-parallel-size 2 --gpu-memory-utilization 0.8 --disable-log-requests --enable-prefix-caching --port 54321 --api-key empty
```

```shell
accelerate launch --multi_gpu --num_machines 1 --num_processes 2 --main_process_port 24599 \
FineTuneLLM.py \
--model_name=meta-llama/Meta-Llama-3-8B-Instruct --adafactor=False \
--tokenizer_name=meta-llama/Meta-Llama-3-8B-Instruct --load_in_4bit \
--victim_model meta-llama/Llama-3.1-8B-Instruct \
--output_dir llama3_rl_finetune/ --batch_size 32 \
--requests_per_minute 100 --target_dataset train_data_pleak.csv \
--epochs 60 --save_freq 4 \
--use_bonus_reawrd True \
--server_url http://localhost:54322/v1 --api_key empty
```

- if you want to use wandb to log the training process, you can add the following arguments:
  `--log_with wandb --wandb_exp_name your_exp_name --wandb_entity your_entity_name`
- The prompt should be stored in `{output_dir}/good_prompts.csv`
- Please choose your own num_process according to your machine's GPU number.

### 3. Evaluate the model

for evaluating the model, you can run the following command:

```shell
# launch vllm for target model
vllm serve meta-llama/Llama-3.1-8B-Instruct --tensor-parallel-size 2 --gpu-memory-utilization 0.8 --disable-log-requests --enable-prefix-caching --port 54321 --api-key empty
```

```shell
# evaluate
python evaluate_task.py \
            --model_name meta-llama/Llama-3.1-8B-Instruct \
            --prompts_data_path /your/output_dir/good_prompts.csv \
            --n_samples 10 \
            --server_url http://localhost:54321/v1 \
            --api_key empty \
            --dataset_path test_data_pleak.csv
```


### 4. Reference
If you find this repository useful, please consider citing the following paper:

```bibtex
@inproceedings{nie2025leakagent,
      title={LeakAgent: RL-based Red-teaming Agent for LLM Privacy Leakage}, 
      author={Yuzhou Nie and Zhun Wang and Ye Yu and Xian Wu and Xuandong Zhao and Wenbo Guo and Dawn Song},
      year={2025},
      booktitle={COLM}, 
}
```