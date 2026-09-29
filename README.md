# LSV-Loc: LiDAR to StreetView Image Cross-Modal Localization

Official implementation of:

**S. Lee, D. Choi, and J.-H. Ryu, "LSV-Loc: LiDAR to StreetView Image Cross-Modal Localization," IEEE Robotics and Automation Letters, 2026.**

LSV-Loc is a cross-modal localization framework that matches LiDAR range images with StreetView images for large-scale vehicle localization. The framework supports multiple LiDAR configurations and learns a shared representation between LiDAR and camera observations for cross-modal place recognition and pose estimation.

## Overview

LSV-Loc provides:

- LiDAR-to-StreetView cross-modal place recognition
- Support for multiple LiDAR configurations
- DINOv2, CLIP, and SEA-lite based feature extractors
- Distributed training with PyTorch DDP
- Place recognition evaluation
- PnP-based metric localization
- Support for multiple public and in-house datasets

## Supported LiDAR Sensors

The current implementation supports range-image generation and evaluation for multiple LiDAR configurations, including:

- Velodyne HDL-32E
- Velodyne HDL-64E
- Ouster OS1-32
- Ouster OS1-64
- Ouster OS2-32
- Ouster OS2-64

## Supported Datasets

The repository includes configurations and utilities for:

- [MulRan](https://sites.google.com/view/mulran-pr/home)
- ComplexUrban
- HeLiPR
- STheReo
- MA-LIO
- In-house datasets

## Repository Structure

```text
LSV-Loc/
├── trainer.py
├── evaluate_PR.py
├── evaluate_VIS.py
├── evaluate_PnP.py
├── requirements.txt
├── setup.py
├── config/
│   ├── strv_config.py
│   └── strv_eval.py
├── utility/
│   ├── Backbone/
│   ├── Database/
│   ├── Network/
│   ├── Eval/
│   └── Etc/
└── result/
```

### Main Components

| Module | Description |
| --- | --- |
| `trainer.py` | Training entry point |
| `evaluate_PR.py` | Place recognition evaluation |
| `evaluate_VIS.py` | Retrieval visualization |
| `evaluate_PnP.py` | PnP-based metric localization evaluation |
| `utility/Backbone` | Feature extraction backbones |
| `utility/Database` | Dataset loaders and preprocessing |
| `utility/Network` | Training losses and network utilities |
| `utility/Eval` | Evaluation functions and metrics |
| `config` | Training and evaluation configurations |

## Environment

The repository is designed for a CUDA-enabled Linux environment.

A Docker environment can be created with:

```bash
sudo docker run -it \
    --name=SVR_Loc_NGC \
    --gpus=all \
    --env="DISPLAY=$DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --env="XAUTHORITY=$XAUTH" \
    --env NVIDIA_DRIVER_CAPABILITIES=compute,utility \
    --volume="$XAUTH:$XAUTH" \
    --runtime=nvidia \
    -v /home/$USER/Workspace_Share/:/home/$USER/Workspace/ \
    -v /home/$USER/Documents/:/home/$USER/Documents/ \
    -v /dev:/dev \
    -v /dev/shm:/dev/shm \
    --privileged \
    --net=host \
    --ipc=host \
    --pid=host \
    -p 8888:8888 \
    iismn/ubuntu_cuda_gl:LTS24-CUDA12-x86
```

## Installation

Clone the repository:

```bash
git clone https://github.com/iismn/LSV-Loc.git
cd LSV-Loc
```

Install the required Python packages:

```bash
pip install -r requirements.txt
```

PyTorch and torchvision should be installed separately according to the CUDA version of the host system.

See the official [PyTorch installation guide](https://pytorch.org/) for details.

## Training

Training parameters are defined in `config/strv_config.py`.

Run training with:

```bash
python trainer.py --train_config strv_config
```

A custom configuration can also be specified:

```bash
python trainer.py --train_config your_custom_config
```

## Evaluation

### Place Recognition

```bash
python evaluate_PR.py
```

### Retrieval Visualization

```bash
python evaluate_VIS.py
```

### PnP Localization

```bash
python evaluate_PnP.py
```

## Configuration

Example training parameters include:

| Parameter | Description | Default |
| --- | --- | ---: |
| `batch_size` | Batch size per GPU | 10 |
| `epochs` | Number of training epochs | 50 |
| `image_size` | Input image resolution | 518 |
| `model_name` | Feature extraction backbone | `match_SEA_lite` |
| `threshold_dist` | Positive distance threshold | 25 m |
| `final_embedding_dim` | Descriptor dimension | 512 |

## Dataset Structure

Datasets are organized under:

```text
dataset/SVR_Dataset_Sync/
```

An example directory structure is:

```text
dataset/SVR_Dataset_Sync/
├── MulRan/
│   ├── DCC01/
│   │   ├── DB/
│   │   │   └── {frame_id}/
│   │   │       └── {frame_id}_Equi.png
│   │   ├── DB_Pos/
│   │   ├── Q/
│   │   │   └── {frame_id}.png
│   │   ├── Q_Pos/
│   │   └── Q_Range/
│   └── KAIST01/
│
├── ComplexUrbanDataset/
│   ├── Urban01/
│   ├── Urban02/
│   ├── Urban13/
│   └── Urban15/
│
├── HeLiPR/
│   ├── Bridge04/
│   ├── Riverside06/
│   ├── Roundabout01/
│   └── Town01/
│
├── STheReo/
│   ├── SNU_Afternoon/
│   └── Valley_Afternoon/
│
├── MA_LIO/
│   ├── City01/
│   ├── City02/
│   └── City03/
│
├── InHouse/
│   ├── ComplexUrbanDataset/
│   ├── Dunsan/
│   ├── KAIST/
│   ├── MulRan/
│   ├── SVR_Test_MiniBatch.mat
│   └── SVR_Train_MiniBatch.mat
│
├── MAT/
│   ├── SVR_Train.mat
│   ├── SVR_Test_All.mat
│   ├── SVR_Test_ComplexUrban05.mat
│   ├── SVR_Test_ComplexUrban08.mat
│   ├── SVR_Test_Dunsan.mat
│   └── SVR_Test_Roundabout01.mat
│
└── Utils/
    ├── streetviewImg_DWL_Main.py
    ├── streetviewImg_DWL_Inhouse.py
    ├── panorama_photo_date_average.py
    └── MATLAB_API/
```

### Data Format

- **Database images (`DB`)**: Equirectangular StreetView images
- **Query images (`Q`)**: LiDAR range images
- **Position files**: Ground-truth or reference positions for database and query samples
- **MAT files**: Training and evaluation indices and metadata

## Citation

If you use this repository in your research, please cite:

```bibtex
@article{LSVLoc,
  author={Lee, Sangmin and Choi, Donghyun and Ryu, Jee-Hwan},
  journal={IEEE Robotics and Automation Letters},
  title={LSV-Loc: LiDAR to StreetView Image Cross-Modal Localization},
  year={2026},
  volume={11},
  number={3},
  pages={2514--2521},
  doi={10.1109/LRA.2026.3653282}
}
```

## Paper

Sangmin Lee, Donghyun Choi, and Jee-Hwan Ryu,  
**LSV-Loc: LiDAR to StreetView Image Cross-Modal Localization**,  
IEEE Robotics and Automation Letters, Vol. 11, No. 3, pp. 2514-2521, 2026.

[IEEE Xplore](https://doi.org/10.1109/LRA.2026.3653282)

## License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.

## Contact

**Sangmin Lee**  
Korea Advanced Institute of Science and Technology (KAIST)

For questions regarding the paper or implementation, please open an issue in this repository.
