# [C3AE]( https://arxiv.org/abs/1904.05059 )

This is a unofficial keras implements of c3ae for age estimation. welcome to discuss ~ 


--------[result]-----------------
<div>
<img src="https://raw.githubusercontent.com/StevenBanama/C3AE/master/assets/example1.jpg" width="200" height="200"><img src="https://raw.githubusercontent.com/StevenBanama/C3AE/master/assets/example2.jpg" width="200" height="200">
</div>

### Update History:

- 2019-9 C3AE org
- 2020-7 transfer to tensorflow2.1 and exposing gender branch.
    - 1.add gender prediction
    - 2.change neck
    - 3.add Mish6, GeM, Smooth label and so on.
    - 4.add utk, afad, asia dataset
    - 5.add tflite freezing
- 2020-8 fixed bugs and add fixed-precision model
    -  1. remove nan weights (it may case predict as unpredict)
    -  2. add -fp16

### Part1 age branch

|source|version|IMDB(mae)|WIKI(mae)|extra change| model|
| -- | -- | :--: | :--: | :--:| :--: |
| from papper | -- | **6.57** | **6.44** | -- | -- |
| our implement | c3ae-v84 | **6.77** | **6.74** | change kl to focal loss without se_net|  model/imdb_focal_loss_c3ae_v84.h5 | model/c3ae_wiki_v87.h5 |
| our implement v2 | c3ae-v89 | **6.58** | -- | SE_NET + focal_loss | model/c3ae_imdb_v89.h5 |
| our implement v3 | c3ae-v90 | **6.51**| -- | white norm + SE_NET + focal_loss | mail to geekpeakspar@gmail.com |


### Part2 add gender branch
Triple-Boxes show much influence with different dataset, meanwhile the distribution plays an important role. 

|source|version| asia| utk| afad | model|
| -- | -- | -- | -- | -- | -- |
| our implement v4 | asia |age: 5.83 gender 0.955 | -- | --| ./model/c3ae_model_v2_117_5.830443-0.955 |
| our implement v4 | asia+utk | -- | age: 5.2 gender 0.967 | --| ./model/c3ae_model_v2_91_5.681206-0.949 |
| our implement v4 | asia+utk+afad |age: 5.9 gender 0.9234 | age: 5.789  gender: 0.9491 | age: 3.61 gender: 0.9827| ./model/c3ae_model_v2_151_4.301724-0.962|
| our implement v4 | fp16+white-norm asia+utk+afad |age: 6.0 gender 0.97 | age: 5.6  gender: 0.942 | age: 3.72 gender: 0.987| ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973|

cation: Gender annotaion of utk is opposite to wiki/imdb/asia.

 **You can change weights of loss to improve age mae, [1, 20, 20].
>> python nets/C3AE_expand.py --white  -se --source "afad" -gpu -m "fp16_white_se" -p ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973 -fp16 -test 

## structs
   - assets 
   - dataset (you`d better put dataset into this dir.)
   - detect (MTCNN and align)
   - download.sh (bash script of downloading dataset)
   - model (pretrain model will be here)
   - nets (all tainging code)
       - C3AE.py 
   - preproccessing (preprocess dataset), which contains "wiki" "imdb" "afad" "asia" "utk"
## Pretrain model(a temp model)
   >> all trainned  model saved in dir named "model"

## required enviroments:
   numpy, tensorflow(2.1), pandas, feather, opencv, python=3.6.5
   
   >>> pip install -r requirements2.1.txt
  
   numpy, tensorflow(1.8), pandas, feather, opencv, python=2.7

   >>> pip install -r requirements.txt

## test
 ### age and gender branch(only for py3 and tensorflow2+)
 - for image
   >>> python nets/test.py -g -white -se -i assets/timg.jpg -m ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973
 - for video
   >>> python nets/test.py -g -white -v -se -m ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973

 ### age branch
 - for image
   >>> python nets/test.py -se -i assets/timg.jpg -m model/c3ae_imdb_v89.h5
 - for video
   >>> python nets/test.py -v -se -m model/c3ae_imdb_v89.h5


##  Preparation Datasets
*download*  imdb/wiki dataset and then *extract* those data to the "./dataset/" \
 [download wiki]( https://data.vision.ee.ethz.ch/cvl/rrothe/imdb-wiki/static/wiki_crop.tar) 
 [download imdb]( https://data.vision.ee.ethz.ch/cvl/rrothe/imdb-wiki/static/imdb_crop.tar)
 [download asia]( https://github.com/JingchunCheng/All-Age-Faces-Dataset)
 [download utk]( https://susanqq.github.io/UTKFace/)
 [download afad]( https://github.com/afad-dataset/tarball)
 

## Preprocess:
    >>> python preproccessing/dataset_proc.py -i ./dataset/wiki_crop --source wiki
    >>> python preproccessing/dataset_proc.py -i ./dataset/imdb_crop --source imdb
    >>> python preproccessing/dataset_proc.py -i ./dataset/AFAD-Full --source afad 

## training: 
    plain net
    >>> python C3AE.py -gpu -p c3ae_v16.h5 -s c3ae_v16.h5 --source imdb -w 10
    with se-net and white-norm (better result)
    >>> python C3AE.py -gpu -p c3ae_v16.h5 -s c3ae_v16.h5 --source imdb -w 10 -white -se
    for gender and age prediction:
    >>> python nets/C3AE_expand.py -se -white --source "afad" -gpu -fp16 -p ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973 
    
## freeze tflite
    python nets/C3AE_expand.py -se -white --source "asia" -gpu -fp16 -p ./model/c3ae_model_v2_fp16_white_se_132_4.208622-0.973 -fz

## DETECT: 
   [mtcnn] (https://github.com/YYuanAnyVision/mxnet_mtcnn_face_detection):  detect\align\random erasing \
   ![trible box](https://raw.githubusercontent.com/StevenBanama/C3AE/master/assets/triple_boundbox.png)

## net struct
![ params ](https://raw.githubusercontent.com/StevenBanama/C3AE/master/assets/params.png) ![ plain_model ](https://raw.githubusercontent.com/StevenBanama/C3AE/master/assets/plain_model.png) 


## Q&A: 
   - only 10 bins in paper: why we got 12 category: we can split it as "[0, 10, ... 110 ]" by two points!\
   - Conv5 1 * 1 * 32, has 1056 params, which mean 32 * 32 + 32. It contains a conv(1 * 1 * 32) with bias 
   - feat: change [4 * 4 * 32] to [12] with 6156 params.As far as known, it may be compose of  conv(6144+12) ,pooling and softmax.
   - the distribution of imdb and wiki are unbalanced, that`s why change the KL loss to focal loss
   - gender prediction: detail in nets/C3AE_expand.py

### To-Do:
    - 1.anchor free boundbox
    - 2.add another new feathers
    
## Reference
  - focal loss: https://github.com/maozezhong/focal_loss_multi_class/blob/master/focal_loss.py
  - mtcnn: https://github.com/YYuanAnyVision/mxnet_mtcnn_face_detection
  
