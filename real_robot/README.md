# Real-Robot Setup and Commands

This file records the main LeRobot commands used during the project. Paths are sanitized so the repository does not expose machine-specific home directories or credentials.

## Local configuration used

- Follower arm: `COM3`
- Leader arm: `COM4`
- Base camera: OpenCV index `0`
- Wrist camera: OpenCV index `2`
- Camera capture: 640×480 at 30 FPS
- Conda environment: `lerobot`

Port and camera assignments can change on another machine.

## Environment

~~~bat
conda activate lerobot
cd <LEROBOT_ROOT>
~~~

## Teleoperation

~~~bat
lerobot-teleoperate --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm
~~~

## Early camera/teleoperation test

This was an early test before the final production camera mapping was settled.

~~~bat
lerobot-teleoperate --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, base: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, rotation: ROTATE_270}}" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true
~~~

## Five-episode pipeline test

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/robotic-arms-so-101_test --dataset.num_episodes=5 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2
~~~

Replay one episode:

~~~bat
lerobot-replay --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --dataset.repo_id=Chenyu3909/robotic-arms-so-101_test --dataset.root="<TEST_DATASET_ROOT>" --dataset.episode=0
~~~

## Record the 25-demo pilot dataset

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.num_episodes=25 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2
~~~

## Train the 25-demo model for 10k updates

~~~bat
lerobot-train --policy.type=act --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_25 --job_name=act_white_bottle_25 --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Fine-tune the 25-demo model for another 10k updates

~~~bat
lerobot-train --policy.path="<LEROBOT_ROOT>\outputs\train\act_white_bottle_25\checkpoints\010000\pretrained_model" --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_25_more10k --job_name=act_white_bottle_25_more10k --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

This loads the previous policy weights but starts a fresh optimizer, so it is documented as **10k + 10k fine-tuning**.

## Append 25 additional demonstrations

This resumed recording into the same local dataset, expanding it from 25 to 50 episodes.

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<DATASET_ROOT>" --dataset.num_episodes=25 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2 --dataset.push_to_hub=false --resume=true
~~~

The local dataset kept its original `pilot-25` name even after it contained 50 episodes.

## Train a fresh 50-demo model for 10k updates

~~~bat
lerobot-train --policy.type=act --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_50 --job_name=act_white_bottle_50 --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Fine-tune the 50-demo model for another 10k updates

~~~bat
lerobot-train --policy.path="<LEROBOT_ROOT>\outputs\train\act_white_bottle_50\checkpoints\010000\pretrained_model" --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_50_more10k --job_name=act_white_bottle_50_more10k --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Autonomous rollout

The rollout command stayed the same across checkpoints; only the policy path changed.

~~~bat
lerobot-rollout --strategy.type=base --policy.path="<POLICY_PATH>" --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --task="Pick up the white bottle and place it in the white box." --duration=0 --device=cuda --display_data=false
~~~

Policy paths used:

| Checkpoint | Relative path from `<LEROBOT_ROOT>` |
| --- | --- |
| 25 demos / 10k | `outputs\train\act_white_bottle_25\checkpoints\010000\pretrained_model` |
| 25 demos / 10k + 10k | `outputs\train\act_white_bottle_25_more10k\checkpoints\010000\pretrained_model` |
| 50 demos / 10k | `outputs\train\act_white_bottle_50\checkpoints\010000\pretrained_model` |
| 50 demos / 10k + 10k | `outputs\train\act_white_bottle_50_more10k\checkpoints\010000\pretrained_model` |

## Security note

Authentication tokens and API keys are deliberately excluded from this repository.
