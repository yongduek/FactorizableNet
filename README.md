# Factorizable Net (F-Net)

This is pytorch implementation of our ECCV-2018 paper: [**Factorizable Net: An Efficient Subgraph-based Framework for Scene Graph Generation**](http://cvboy.com/publication/eccv2018_fnet/). This project is based on our previous work: [**Multi-level Scene Description Network**](https://github.com/yikang-li/MSDN). 

## Progress
- [x] Guide for Project Setup
- [x] Guide for Model Evaluation with pretrained model
- [x] Guide for Model Training
- [x] Uploading pretrained model and format-compatible datasets.
- [ ] A demonstration of our Factorizable Net 


## Project Settings

1. Install the requirements (you can use pip or [Anaconda](https://www.continuum.io/downloads)):

    ```
    conda install pip pyyaml sympy h5py cython numpy scipy
    conda install -c menpo opencv3
    conda install -c soumith pytorch torchvision cuda80 
    pip install easydict
    ```

2. Clone the Factorizable Net repository
    ```bash
    git clone git@github.com:yikang-li/FactorizableNet.git
    ```

3. Build the Cython modules for nms, roi pooling,roi align modules
    ```bash
    cd lib
    ./make.sh
    cd ..
    ```
5. Download the three datasets [**VG-MSDN**](https://drive.google.com/open?id=1WjetLwwH3CptxACrXnc1NCcccWUVDO76), [**VG-DR-Net**](https://drive.google.com/open?id=1WjetLwwH3CptxACrXnc1NCcccWUVDO76), [**VRD**](https://drive.google.com/open?id=12oLtVSCEusG7tG4QwxeJEDsVhiE9gb2s) to ```F-Net/data```. And extract the folders with ```tar xzvf ${Dataset}.tgz```. We have convert to original annotations to ```json``` version. 

6. Download [**Visual Genome images**](http://visualgenome.org/api/v0/api_home.html) and [**VRD**](http://imagenet.stanford.edu/internal/jcjohns/scene_graphs/sg_dataset.zip) images. 
7. Link the image data folder to 	target folder: ```ln -s /path/to/images F-Net/data/${Dataset}/images```
	- p.s. You can change the default data directory by modifying ```dir``` in ```options/data_xxx.json```.
8. [optional] Download the pretrained RPN for [Visual Genome](https://drive.google.com/open?id=12oLtVSCEusG7tG4QwxeJEDsVhiE9gb2s) and [VRD](https://drive.google.com/open?id=1OdzZKn5ZBIXFdxeOjCvjqNhFjobWnDS9). Place them into ```output/```.
4. [optional] Download the pretrained Factorizable Net on [VG-MSDN](https://drive.google.com/open?id=1W7PYyYvkROzC_GZwrgF0XS4fH6r2NyyV), [VG-DR-Net](https://drive.google.com/open?id=1ROWPoMC7A2a-j0v5YqPbpRWQdaNcaRuF) and [VG-VRD](https://drive.google.com/open?id=1n-8d4K7-PywVwuA90x50nnIW1TKyKHU4), and place them to ```output/trained_models/```

## Project Organization
There are several subfolders contained:

- ```lib```: dataset Loader, NMS, ROI-Pooling, evaluation metrics, etc. are listed in the folder.
- ```options```: configurations for ```Data```, ```RPN```, ```F-Net``` and ```hyperparameters```.
- ```models```: model definations for ```RPN```, ```Factorizable``` and related modules.
- ```data```: containing VG-DR-Net (```svg/```), VG-MSDN (```visual_genome/```) and VRD (```VRD/```).
- ```output```: storing the trained model, checkpoints and loggers.

## Evaluation with our Pretrained Models
Pretrained models on [VG-MSDN](https://drive.google.com/open?id=1iKgYVLTUHi_VpmWrQJ6o1OMj3aGlrmDC), [VG-DR-Net](https://drive.google.com/open?id=1ROWPoMC7A2a-j0v5YqPbpRWQdaNcaRuF) and [VG-VRD](https://drive.google.com/open?id=1n-8d4K7-PywVwuA90x50nnIW1TKyKHU4) are provided. ```--evaluate``` is provided to enable evaluation mode. Additionally, we also provide ```--use_gt_boxes``` to fed the ground-truth object bounding boxes instead of RPN proposals. 

- Evaluation on **VG-MSDN** with pretrained.
Scene Graph Generation results:  Recall@50: ```12.984%```, Recall@100: ```16.506%```.

```
CUDA_VISIBLE_DEVICES=0 python train_FN.py --evaluate --dataset_option=normal \
	--path_opt options/models/VG-MSDN.yaml \
	--pretrained_model output/trained_models/Model-VG-MDSN.h5
```



- Evaluation on **VG-VRD** with pretrained. :  Scene Graph Generation results:  Recall@50: ```19.453%```, Recall@100: ```24.640%```.

```
CUDA_VISIBLE_DEVICES=0 python train_FN.py --evaluate \
	--path_opt options/models/VRD.yaml \
	--pretrained_model output/trained_models/Model-VRD.h5
```

- Evaluation on **VG-DR-Net** with pretrained.
Scene Graph Generation results:  Recall@50: ```12.984%```, Recall@100: ```16.506%```.

```
CUDA_VISIBLE_DEVICES=0 python train_FN.py --evaluate --dataset_option=normal \
	--path_opt options/models/VG-DR-Net.yaml \
	--pretrained_model output/trained_models/Model-VG-DR-Net.h5
```


## Training
- Training Region Proposal Network (RPN). The **shared conv layers** are fixed. We also provide pretrained RPN on [Visual Genome](https://drive.google.com/open?id=1W7PYyYvkROzC_GZwrgF0XS4fH6r2NyyV) and [VRD](https://drive.google.com/open?id=1OdzZKn5ZBIXFdxeOjCvjqNhFjobWnDS9). 
	
	```
	# Train RPN for VG-MSDN and VG-DR-Net
	CUDA_VISIBLE_DEVICES=0 python train_rpn.py --dataset_option=normal 
	
	# Train RPN for VRD
	CUDA_VISIBLE_DEVICES=0 python train_rpn_VRD.py 
	
	```

- Training Factorizable Net: detailed training options are included in ```options/models/```.

	```
	# Train F-Net on VG-MSDN:
	CUDA_VISIBLE_DEVICES=0 python train_FN.py --dataset_option=normal \
		--path_opt options/models/VG-MSDN.yaml --rpn output/RPN.h5
		
	# Train F-Net on VRD:
	CUDA_VISIBLE_DEVICES=0 python train_FN.py  \
		--path_opt options/models/VRD.yaml --rpn output/RPN_VRD.h5
		
	# Train F-Net on VG-DR-Net:
	CUDA_VISIBLE_DEVICES=0 python train_FN.py --dataset_option=normal \
		--path_opt options/models/VG-DR-Net.yaml --rpn output/RPN.h5
	
	```
	
	```--rpn xxx.h5``` can be ignore for end-to-end training from pretrained **VGG16**. Sometime, unexpected and confusing errors appear. *Ignore it and restart to training.*
	
- For better results, we usually re-train the model with additional epochs by resuming the training from the checkpoint with ```--resume ckpt```:

	```
	# Resume F-Net training on VG-MSDN:
	CUDA_VISIBLE_DEVICES=0 python train_FN.py --dataset_option=normal \
		--path_opt options/models/VG-MSDN.yaml --resume ckpt --epochs 30
	```

## Acknowledgement

We thank [longcw](https://github.com/longcw/faster_rcnn_pytorch) for his generous release of the [PyTorch Implementation of Faster R-CNN](https://github.com/longcw/faster_rcnn_pytorch). 


## Reference

If you find our project helpful, your citations are highly appreciated:

@inproceedings{li2017msdn,  
	author={Li, Yikang and Ouyang, Wanli and Zhou, Bolei and Wang, Kun and Wang, Xiaogang},  
	title={Scene graph generation from objects, phrases and region captions},  
	booktitle = {Proceedings of the IEEE International Conference on Computer Vision},  
	year      = {2017}  
}

@inproceedings{li2018fnet,  
	author={Li, Yikang and Ouyang, Wanli and Zhou, Bolei and Jianping, Shi and Chao, Zhang and Wang, Xiaogang},  
	title={Scene graph generation from objects, phrases and region captions},  
	booktitle = {Proceedings of the IEEE International Conference on Computer Vision},  
	year      = {2017}  
}


## License:

The pre-trained models and the Factorizable Network technique are released for uncommercial use.

Contact [Yikang LI](http://www.cvboy.com/) if you have questions.