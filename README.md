# MTCNN

Repository for "[Joint Face Detection and Alignment using Multi-task Cascaded Convolutional Neural Networks](https://arxiv.org/abs/1604.02878)", implemented with Caffe, C++ interface. Compared with Cascade CNN, MTCNN integrates the detection net and calibration net into one net. Moreover, face alignment is also applied in the same net.

## Result

### Face Detection
The results of each procedure in MTCNN are contained in result folder. The final results are shown in the flowing. These two pictures are collected by FDDB separately. The MTCNN fails to detect a face which is contained in FDDB in the left picture, while it detect a face in the right picture which is not contained in FDDB. (Better than the benchmark in some cases.)
<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/result/img_838-compare.jpg" width="400">
<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/result/img_906-compare.jpg" width="330">

### Face Alignment

<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/result/O-Net_nms.jpg" width="1000">

### Time Cost
The average time cost is faster than Cascade CNN which is 0.197 s/frame. The result is generated by testing a 1080P live video.

<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/result/result_gpu_2.png" width="1000">

### Accuracy

The accuracy in FDDB which is higher than 0.9. The model contained in  as a pretrain model and improve the result

<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/result/FDDB_evaluation/discroc_MTCNN_20.png" width="1000">

## How to train

A hdf5 dataset is necessray to train a model which has multiple labels in a Caffe model. A sample of script which can generate .hdf5 file is list [here](https://github.com/foreverYoungGitHub/MTCNN/blob/master/script/train_sh/generate_hdf5.py).

In this sample, you need to prepare 4 txt file, which contains label (0 or 1), landmark (ratio in the cropped image), regression box (ratio), and cropped image pathes. A database contained landmarks infomation is needed to generate the sample with these multiple attribute, such as [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html). Then change the pathes in the sample code. The output hdf5 file is shown in `train_file_path`. Notice the image size needs to be changed to generate suitable data for you net.

```
label_path = '../dataset/label.txt'
landmark_path = '../dataset/landmark.txt'
regression_box_path = '../dataset/regression_box.txt'
crop_image_path = '../dataset/crop_image.txt'
train_file_path = '../dataset/train_24.hd5'
```

Then, write the path of the hdf5 file into a txt and contain the txt in prototxt file. 

```
hdf5_data_param {
   source: "/Users/Young/Documents/Programming/MTCNN/MTCNN_train/test_48.txt"
   batch_size: 100
 }
```

Run the shell file and train the model.

### Train strategy

The images are used to train the CNN divide into 3 groups: positive face, negative face and part of the face. When the Intersection-over-Union (IoU) ratio of the cropped image is higher than 0.7, lower than 0.3, or between 0.3 and 0.7 to any ground truth provided by dataset, it belongs to the positive face, the negative face or the part of the face. (You can decide the thresholds by yourself.) The general train process is shown in following figure, while the detialed train processes are also listed.

<img src="https://github.com/foreverYoungGitHub/MTCNN/blob/master/script/train_process.png" width="600">

1. Cropping the images from the dataset randomly, and dividing into positive face, negative face and part of the face based on the IoU between the ground truth and cropped image;
2. Training the P-Net based on the randomly cropped image;
3. Cropping the image from the dataset based on the detected bounding box of the P-Net, dividing it and utilizing it to fine-tuning the P-Net;
4. Training the R-Net based on the data used to fine-tuning the P-Net;
5. Cropping the image from the dataset based on the detected bounding box of the R-Net, dividing it and utilizing it to fine-tuning the R-Net;
6. Training the O-Net based on the data used to fine-tuning the P-Net and R-Net;
7. Cropping the image from the dataset based on the detected bounding box of the O-Net, dividing it and utilizing it to fine-tuning the O-Net.

### The example label for the 3 types data:

positive face:
   - face detection: 1;
   - face landmark: [0.1,0.2,0.3,0.4,0.5,0.1,0.2,0.3,0.4,0.5];
   - face regression: [0.1,0.1,0.1,0.1].
part of the face:
   - face detection: 1;
   - face landmark: [0.1,0.2,0.3,0.4,0.5,0.1,0.2,0.3,0.4,0.5];
   - face regression: [0.4,0.4,0.4,0.4].
negative face:
   - face detection: 0;
   - face landmark: [0,0,0,0,0,0,0,0,0,0];
   - face regression: [0,0,0,0].
   
You can also train the face detection and regression for the dataset without landmark label. The model is then used to train the face landmark.
