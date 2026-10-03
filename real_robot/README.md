# Real-Robot Setup and Commands

These are the main LeRobot commands used during the project.

## Important notes

- Commands were run on Windows in a Conda environment named lerobot.
- Local serial-port assignments were COM3 for the follower and COM4 for the leader.
- Camera indices can change after reconnecting USB devices.
- Machine-specific dataset cache paths are replaced below with placeholders.
- **Authentication tokens and API keys are intentionally omitted. Never commit them to Git.**

## Environment

~~~bat
conda activate lerobot
cd C:\Users\cheny\lerobot
~~~

## Teleoperation only

~~~bat
lerobot-teleoperate --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm
~~~

## Early camera test

This was an early camera/teleoperation test. The final dataset configuration used the production camera mapping shown later.

~~~bat
lerobot-teleoperate --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, base: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, rotation: ROTATE_270}}" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true
~~~

## Production camera mapping

Final recording and rollout configuration:

- base camera: index 0
- wrist camera: index 2
- resolution: 640×480
- frame rate: 30 FPS

## Five-episode test recording

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/robotic-arms-so-101_test --dataset.num_episodes=5 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2
~~~

## Replay a recorded episode

~~~bat
lerobot-replay --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --dataset.repo_id=Chenyu3909/robotic-arms-so-101_test --dataset.root="<TEST_DATASET_ROOT>" --dataset.episode=0
~~~

## Record 25-demo pilot dataset

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.num_episodes=25 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2
~~~

## Train 25-demo ACT policy for 10k updates

~~~bat
lerobot-train --policy.type=act --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<PILOT_DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_25 --job_name=act_white_bottle_25 --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Roll out 25-demo / 10k policy

~~~bat
lerobot-rollout --strategy.type=base --policy.path="C:\Users\cheny\lerobot\outputs\train\act_white_bottle_25\checkpoints\010000\pretrained_model" --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --task="Pick up the white bottle and place it in the white box." --duration=0 --device=cuda --display_data=true
~~~

## Roll out 25-demo / 10k + 10k fine-tuned policy

~~~bat
lerobot-rollout --strategy.type=base --policy.path="C:\Users\cheny\lerobot\outputs\train\act_white_bottle_25_more10k\checkpoints\010000\pretrained_model" --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --task="Pick up the white bottle and place it in the white box." --duration=0 --device=cuda --display_data=false
~~~

## Append 25 additional demonstrations

This resumed recording into the same local dataset, expanding it from 25 to 50 episodes.

~~~bat
lerobot-record --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=leader_arm --display_data=true --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<PILOT_DATASET_ROOT>" --dataset.num_episodes=25 --dataset.single_task="Pick up the white bottle and place it in the white box." --dataset.streaming_encoding=true --dataset.rgb_encoder.vcodec=h264 --dataset.encoder_threads=2 --dataset.push_to_hub=false --resume=true
~~~

## Train fresh 50-demo policy for 10k updates

~~~bat
lerobot-train --policy.type=act --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<PILOT_DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_50 --job_name=act_white_bottle_50 --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Fine-tune 50-demo model for an additional 10k updates

This initialized from the 50-demo / 10k policy weights with a fresh optimizer.

~~~bat
lerobot-train --policy.path="C:\Users\cheny\lerobot\outputs\train\act_white_bottle_50\checkpoints\010000\pretrained_model" --dataset.repo_id=Chenyu3909/so101-white-bottle-pilot-25 --dataset.root="<PILOT_DATASET_ROOT>" --output_dir=outputs/train/act_white_bottle_50_more10k --job_name=act_white_bottle_50_more10k --policy.device=cuda --wandb.enable=false --policy.push_to_hub=false --steps=10000
~~~

## Roll out 50-demo / 10k policy

~~~bat
lerobot-rollout --strategy.type=base --policy.path="C:\Users\cheny\lerobot\outputs\train\act_white_bottle_50\checkpoints\010000\pretrained_model" --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --task="Pick up the white bottle and place it in the white box." --duration=0 --device=cuda --display_data=false
~~~

## Roll out 50-demo / 10k + 10k fine-tuned policy

~~~bat
lerobot-rollout --strategy.type=base --policy.path="C:\Users\cheny\lerobot\outputs\train\act_white_bottle_50_more10k\checkpoints\010000\pretrained_model" --robot.type=so101_follower --robot.port=COM3 --robot.id=follower_arm --robot.cameras="{ base: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, wrist: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30} }" --task="Pick up the white bottle and place it in the white box." --duration=0 --device=cuda --display_data=false
~~~

## Local dataset note

The dataset directory retained its original pilot-25 name after the additional 25 episodes were appended. The final local dataset contains **50 episodes even though the repository/folder name still references 25**.
