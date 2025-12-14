#
## Create a symlink so [train.py](./scripts/imitation_learning/robomimic/train.py) finds [`robomimic`](../scripts/imitation_learning/robomimic)

Validate the import robomimic
```bash
./isaaclab.sh -p -c "import robomimic; print('robomimic import OK')"
```
If error appeared:

```bash
cd source/isaaclab_tasks/isaaclab_tasks/manager_based/manipulation/pick_place
ln -sfn agents/robomimic robomimic
```


Get the annotated Dataset
```bash
curl -L --fail --retry 5 --retry-delay 2   -o datasets/dataset_annotated_gr1.hdf5   https://omniverse-content-production.s3-us-west-2.amazonaws.com/Assets/Isaac/5.1/Isaac/IsaacLab/Mimic/pick_place_datasets/dataset_annotated_gr1.hdf5
```
- `dataset_annotated_gr1.hdf5` is the starting dataset (annotated demonstrations) used by the Mimic pipeline.



Step converts/expands the annotated dataset into a generated dataset used for training
```bash
./isaaclab.sh -p scripts/imitation_learning/isaaclab_mimic/generate_dataset.py   --device cpu   --headless   --enable_pinocchio   --num_envs 20   --generation_num_trials 1000   --input_file ./datasets/dataset_annotated_gr1.hdf5   --output_file ./datasets/generated_dataset_gr1.hdf5
```
- `--device cpu`: runs dataset generation on CPU (useful for machines without a configured GPU).
- `--headless`: no rendering window.
- `--num_envs 20`: number of parallel simulated environments.
- `--generation_num_trials 1000`: how many trials to attempt when generating.
- `--enable_pinocchio`: enables Pinocchio-based kinematics (often needed for consistent control/IK behavior).

Output:
- `datasets/generated_dataset_gr1.hdf5`


Begin Training
```bash
./isaaclab.sh -p scripts/imitation_learning/robomimic/train.py \
  --task Isaac-PickPlace-GR1T2-Abs-v0 \
  --algo bc \
  --normalize_training_actions \
  --dataset ./datasets/generated_dataset_gr1.hdf5
```

- `--algo bc` trains a supervised policy to mimic actions from demonstrations.
- `--normalize_training_actions` normalizes action targets during training

Get <NORM_FACTOR_MIN> and <NORM_FACTOR_MAX>
```bash
RUN_DIR=$(ls -td logs/robomimic/Isaac-PickPlace-GR1T2-Abs-v0/bc_rnn_low_dim_gr1t2/* | head -1)
cat "$RUN_DIR/logs/normalization_params.txt"
```
```bash
cat logs/robomimic/Isaac-PickPlace-GR1T2-Abs-v0/bc_rnn_low_dim_gr1t2/20251213213603/logs/normalization_params.txt
```


Run policy from the normalization factor 
```bash
./isaaclab.sh -p scripts/imitation_learning/robomimic/play.py   --device cpu   --enable_pinocchio   --task Isaac-PickPlace-GR1T2-Abs-v0   --num_rollouts 10   --horizon 400   --norm_factor_min <NORM_FACTOR_MIN>   --norm_factor_max <NORM_FACTOR_MAX>   --checkpoint /PATH/TO/desired_model_checkpoint.pth
```
- `--num_rollouts`: how many episodes to run.
- `--horizon`: max steps per rollout.
- Checkpoints are typically saved inside the run directory under a `checkpoints/` folder 


<!-- <p align="center">
  <img src="images/gif-PickPlace-GR1T2-Abs-v0.gif" alt="Subtask splitting example" width="2070">
</p>
 -->


<p align="center">
  <img src="images/demo.gif" alt="Subtask splitting example" width="970">
</p>


