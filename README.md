# VATT (Video-to-Audio Generation Through Text)
[Xiulong Liu](https://dragonliu1995.github.io/), [Kun Su](https://kun-su.netlify.app/), [Eli Shlizerman](http://faculty.washington.edu/shlizee/)






Official repository that contains **code, datasets, and sample outputs** for [NeurIPS paper](#citation)  **"Tell What You Hear From What You See — Video to Audio Generation Through Text"**, accepted as poster in NeurIPS 2024.

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)  
3. [Datasets](#datasets)
4. [Code](#code)
5. [Checkpoints](#checkpoints)
6. [Inference-Instructions](#inference-instructions)
7. [Training-Instructions](#training-instructions)
8. [Samples-and-Demo](#samples-and-demo)
9. [Citation](#citation)
10. [Contact](#contact)

## Introduction
**VATT (Video-to-Audio Generation Through Text)** is a multi-modal generative framework in **pure discrete space** that takes a video and an optional text prompt as input, and generates audio and optional textual description of the audio. Such a framework has two advantages: i) Video-to-Audio generation process can be refined and controlled via text which complements the context of visual information, and ii) The model can suggest what audio to generate for the video by generating audio captions.

<p align="center">
<img src="https://github.com/user-attachments/assets/1173211a-258f-49e4-a222-6c76f3c5548e" width="700" />
</p>



VATT consists of two key modules: **VATT Converter**, a LLM that is fine-tuned for instructions and includes a projection layer that maps video features to the LLM vector space; and **VATT Audio**, a transformer that generates audio tokens from visual frames and from optional text prompt using iterative parallel decoding. The audio tokens are converted to a waveform by pretrained neural codec. 

<p align="center">
<img src="https://github.com/user-attachments/assets/a05c81b5-35dd-4c30-b2e6-95210a16a2a2" width="1000" />
</p>



Check out these demos videos (both real and Sora generated videos), now featuring audio generated by **VATT**:



https://github.com/user-attachments/assets/fb176a8c-88f8-437c-9c2a-b9e9f9bf559c




https://github.com/user-attachments/assets/6ae561af-ff3c-4b3e-8e7d-9e38c1765799





https://github.com/user-attachments/assets/f0a2ec41-c6b5-49a2-9d33-cae4fba464af




https://github.com/user-attachments/assets/e37be3ab-a694-4d3a-8d81-d41ba05c934c






✨ If you find this repository helpful for your research, please give us a star ⭐ and consider citing our paper (see [Citation](#citation)).


## Installation
For ease of usage, just follow the commands in ``setup_inference.sh`` to set up the environment for VATT inference. The installation steps assume Linux environment. Also, we assume that the inference runs using GPU with cuda. And this script successfully runs on a single A100 40GB GPU.

## Datasets
Here, we provide links to our **V2A Instruction Dataset** and **extracted video and audio features from visual and audio encoders**.
### V2A Instruction Dataset
1. **VGGSound Only**  
   [Download Link](https://drive.google.com/file/d/1uo4Hx6tAnqVkU65AfPHGwFAftysTCXxs/view)  
   This subset consists solely of VGGSound data.

2. **VGGSound + AudioSet 2M**  
   [Download Link](https://drive.google.com/file/d/1ukpU69eysXnhrHOfgSVWf2BHE5E4WuzI/view)  
   This expanded version includes both VGGSound and some additional data from AudioSet, totaling 1.77 million samples.

**Extracted eva-CLIP features (5 fps) from VGGSound videos**
[Download Link](https://drive.google.com/file/d/1Mgb1CWNqL99q4DWh57derAfDdQeOEkBp/view?usp=drive_link) 

**Extracted audio tokens from VGGSound audio using Encodec-16kHz**
[Download Link](https://drive.google.com/file/d/1_dYe52NcsG0fkvgMa4ixaFQtj1Xwqovv/view?usp=drive_link)

> **Note:** Please check the appropriate licenses and usage rights for VGGSound and AudioSet data before using them in your research.

## Code
We include our full code implementations in vatt folder, including both stages: video-to-caption stage (v2cap) and video+text->audio stage (vt2a). Instructions for both training and inference are detailed below. 

## Checkpoints
**VATT Full Models (including LLama and Gemma version, 4 checkpoints in total in the zip file)**
[Download Link](https://www.dropbox.com/scl/fi/o1663dvgavdtgryltqvmq/vatt_models.zip?rlkey=yzk7o6fb6kpdvs7mnke03t3c0&st=wr75y7xn&dl=0)

**Full AudioGen-Encodec model checkpoint being used for converting tokens back to audio waveform**
[Download Link](https://www.dropbox.com/scl/fi/48nasacz1qikulwv4nj87/audiogen_models.zip?rlkey=3j4beimh2q8h9rw3jmd1mxqsd&st=lbv24b4c&dl=0)

**Vicuna pretrained checkpoint and finetuned delta weights**
[Download Link](https://drive.google.com/file/d/1rjg-_DKzpxCxX51gwiFD3vYv3iKBPQNM)
[Download Link](https://drive.google.com/file/d/1Z-R72RXCapiWUcc35qq8vkzv7U9jrCKu)

**Gemma pretrained checkpoint and finetuned delta weights**
[Download Link](https://www.dropbox.com/scl/fi/uvq21ybjnhkj3uv1xik7b/gemma-finetuned-checkpoint-231000.zip?rlkey=2x077622qaz5jsgkt2dq4diyj&st=93zcldbf&dl=0)
[Download Link](https://www.dropbox.com/scl/fi/z1x100rbp6fim0n54hi7e/gemma_2b.zip?rlkey=2q04foi9kmu727cbxexs56t6h&st=09517hbo&dl=0)

**Pretraiend Encodec codebooks**
[Download Link](https://drive.google.com/file/d/1LdGOtB1s91Lc6Gi45ZD9cSZp6-1qHhLP)

## Inference-Instructions
(**You can skip this if you already follow the ``setup_inference.sh`` to perform inference, this section is a more detailed explanation.**)

Below we show setup of VATT-LLama-T inference, other variants of VATT inference are similar.

1. Set up the environment (The project has been tested on Linux enviroment):
   - create a conda environment with Python 3.10 with torch version 2.0.1 and cuda 11.7:
      ```bash
         conda create --prefix vatt_env python=3.10
         conda activate vatt_env
         pip install torch==2.0.1+cu117 torchvision==0.15.2+cu117 torchaudio==2.0.2+cu117 --extra-index-url https://download.pytorch.org/whl/cu117
      ```
   -  Go to ``vatt`` folder, and run ``pip install -r requirements.txt``.
   -  Go to the folder "third_party/hf-dev/transformers-main" and "third_party/peft-main" respectively to install modified version of these two libraries (required by vatt) using "pip install -e ."

2. Go to the folder vatt/vt2a and add PYTHONPAPTH by "export PYTHONPATH=/path/to/vatt:$PYTHONPATH".  Run the script "vt2a_mlm_ff_generation.py" to generate the audio tokens and save to a folder of your choice. Before running, make sure to configure all required paths of model checkpoints and data (those occurences of "/path/to/" in vt2a_mlm_ff_generation.py) for correct paths. Also, make sure to configure required paths in the yaml config file in ``configs/vt2a_mlm_alibi_mix_large_unicodec_vgg_stage_2.yaml''.

3. Run the script "encodec2audio.py" to recover the audio waveform from the audio tokens. This step requires installation of audiocraft. We reccommend that you create a separate conda env and follow installation guidelines in https://github.com/facebookresearch/audiocraft to set up the environment for decoding audio tokens using Encodec-16kHz.

Note: Above is the inference procedure for VGGSound videos, where we already preprocessed the video features, i.e., eva-CLIP features at 5fps rate, for you. If you do want to try VATT inference for videos other than VGGSound dataset, you need to first exract these video features at 5fps rate, and then use open_clip library (See [link](https://github.com/mlfoundations/open_clip)) combined with the following eva-CLIP checkpoint (See [link](https://huggingface.co/timm/eva02_large_patch14_clip_336.merged2b_s6b_b61k)) to obtain the video features. We provide an example script [here](https://www.dropbox.com/scl/fi/g8su93q8ntm5yk4m24n2p/extract_vggs_clip_5fps.py?rlkey=zaxyhajxc30544mradeepjdo6&st=k1l0t412&dl=0) for you to extract the corresponding video features for each video frame. 

## Training-Instructions

1. Video-to-Audio Captioning Stage (V2ACap): We follow the general instruction tuning procedure of MLLMs to train the VATT Converter in two sub-stages. Take VATT-LLama Converter as example. In the first sub-stage, we tune only the visual projection layer of the MLLM. The training script is provided at the path ```vatt/v2cap/train_script/finetune_v2a_proj_only.sh```. After the first sub-stage, we then perform LoRA finetuning along with the visual projector. The training script is provided at the path ```vatt/v2cap/train_script/finetune_v2a_llama.sh```.

2. Once the first stage training is done, we freeze all weights for the MLLM, and then train the audio generator, VATT Audio. For VATT Audio, we also train in two substages. In the first sub-stage, we use dummy instructions as text inputs (without actual text description), e.g.,  "imagine possible audio for this video". Run the script
   ```bash
   srun python3 vatt/vt2a/vt2a_mlm_train.py -b vatt/vt2a/configs/vt2a_mlm_alibi_mix_large_unicodec_vgg_stage.yaml -l stage_1_logs/
   ```
   Once you train for sufficient steps, we switch over to the training using mixture of dummy instructions and actual audio description as text prompt. To do so, Go to ```modules/vt2a_mlm_alibi_uni_encodec.py```, change ckpt_path in line 55 from None to the path to the best checkpoint in stage 1 training. Then run the following script: 
   ```bash
   srun python3 vatt/vt2a/vt2a_mlm_train.py -b vatt/vt2a/configs/vt2a_mlm_alibi_mix_large_unicodec_vgg_stage_2.yaml -l stage_2_logs/
   ```

## Samples-and-Demo
We provide sample outputs generated by **VATT-LLama-T** on the VGGSound test set:

- **VATT-LLama-T (VGGSound Test Set)**  [Download Link](https://drive.google.com/file/d/10DVuVOxn_2eDUdSYLrtB0XSkkCgJMY3a/view?usp=sharing)

We also provide demos of generated audio from videos comparing against prior methods, please visit  [Demo Website](https://dragonliu1995.github.io/VATT/) for more details.

## Citation
If you use VATT or refer to our NeurIPS paper, please cite us:

```bibtex
@article{liu2024tell,
  title={Tell What You Hear From What You See--Video to Audio Generation Through Text},
  author={Liu, Xiulong and Su, Kun and Shlizerman, Eli},
  journal={arXiv preprint arXiv:2411.05679},
  year={2024}
}
```

Or

```
@inproceedings{NEURIPS2024_b782a346,
 author = {Liu, Xiulong and Su, Kun and Shlizerman, Eli},
 booktitle = {Advances in Neural Information Processing Systems},
 editor = {A. Globerson and L. Mackey and D. Belgrave and A. Fan and U. Paquet and J. Tomczak and C. Zhang},
 pages = {101337--101366},
 publisher = {Curran Associates, Inc.},
 title = {Tell What You Hear From What You See - Video to Audio Generation Through Text},
 url = {https://proceedings.neurips.cc/paper_files/paper/2024/file/b782a3462ee9d566291cff148333ea9b-Paper-Conference.pdf},
 volume = {37},
 year = {2024}
}
```

## Contact
For any questions regarding the code, model checkpoints and dataset, please reach out directly to [email](liuxiulong1995@gmail.com)
