# CVPR2015-DL-for-Person-Re-Identification
Implementation of Deep Learning Architecture for Person Re-Identification, in CVPR2015.

# Enviroment
Packages utilized in this project including:  
* python 3.5.4  
* opencv 3.1.0  
* matplotlib 2.0.2  
* numpy 1.12.1
* keras 2.1.5

# Dataset
The current model was trained on CUHK01 dataset which consists of 971 identities, with 2 images per person in each view. You can check for more details on http://www.ee.cuhk.edu.hk/~xgwang/CUHK_identification.html.<br>
More specifically, we take as input cropped and augmented pictures instead of raw photos, which we will discuss later in training process.

# Overall Architecture
As represented explicitly in the paper, the model is composed of tied convolution layers and higher layers that compute relationships between two pictures, sharing similar attributes with Siamese neural networks.<br>
You can probably gain more intution by referring to related work, or check out `model.py`.
<br>
<br>
![](https://github.com/AlanXia0118/Resource/blob/master/DL-for-ReID/model.png)
<br>
<br>

# Visualized prediction
`predict_visualization.py` is prepared for predicting on your own pairs of identities, employing opencv and matplotlib packages to realize the visualization. You can change the paths to be your model and image at the start of the script:

```
# predict on your own pairs of identities
img_path1 = './test_dataset/9_1.png'
img_path2 = './test_dataset/8_1.png'
```
A pre-trained model was kept for you in `~/model`. Since we trained the model using normalized data, mean images of dataset are required for predicting. Accordingly, one mean image for each channel is provided in `~/mean_img`. You should see a visualized prediction as below:
<br>
<br>
![](https://github.com/AlanXia0118/Resource/blob/master/DL-for-ReID/pre_same.png)
<br>
<br>
Moreover, you are welcome to predict on pictures we collected and stored in `~/test_dataset`.



# Training and validation
The training process is executed in `model_train_and_val.py` as well as validation. Since the model is overall end-to-end, you can start training by feeding two channels of inputs to the first layer just like any other typical cnn architecture.

However, according to the paper, we firstly implement data augmentation and hard negative mining in `npydata_generator.py`, which produces ndarray datasets catering to Keras architectures, to acquire sufficient data and address the problem of imbalanced dataset. 

* Data Augmentation<br>
From 971 original identities, with 2 images per person in each view in CUHK01, due to the protocol that it is better suited for deep learning to use 90% of the data for training, we divided the dataset and took 871 identities for training process. By defining and calling method `translate_and_crop()`, we sample 5 images around the image center with given translation range`[-0.05H, 0.05H]×[-0.05W, 0.05W]`, which was depicted exactly in the paper. Then we utilize `label_positive()` to label a batch of augmented(i.e. newly sampled) picture pairs postive. 
```
label_positive(img_channel1=img_channel1_train,
                   img_channel2=img_channel2_train,
                   labels=labels_train,
                   paths=train_path)
```
There is actullay flexibility in how to feed pairs of images to 2 different channels, and here the strategy we employ is to ensure symmetry so that each channel should be robust to the uncertainty of input view. The final size of positive pairs should be 31356, as detailed computation was shown in the `npydata_generator.py`.

* Hard Negative Mining<br>
Data augmentation increases the number of positive pairs, but imbalance in dataset may still be possibly invited by directly generating negative pairs from all raw images. Therefor, we randomly group the dataset(e.g. divide 871 identities into 13 groups of 67 identities) and label each pair inside negative, and manage to maintain the overall size of negative examples twice as many as postive examples as emphasized in the referrence paper.

Due to several randomly sampling operations, evertime running the `npydata_generator.py` should produce different datasets. 

