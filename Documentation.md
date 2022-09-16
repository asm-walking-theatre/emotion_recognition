# Documentation
## Libraries Used
* tensorflow
* keras
* mlxtend  [(resource link)](http://rasbt.github.io/mlxtend/user_guide/image/extract_face_landmarks/#overview)
* openCV

## Datasets Used
* JAFFE   : No of images: 213 [Link to dataset](https://www.kaggle.com/andrewmvd/japanese-female-facial-expression-dataset-ja)
* Ck-plus : No of images: 981 [Link to dataset](https://www.kaggle.com/shawon10/ckplus)

## Steps implemented  [Link of implemented Research Paper](https://link.springer.com/article/10.1007/s00371-019-01627-4)
## `Image Loading` 
_Function_ : `load_dataset(img_dir,emotion_code)`<br>
**To read all the images of the dataset and convert them into an array**<br>

**_Input_** : <br>
_img_dir_ -- path to the images folder<br>
_emotion_code_ -- list containing emotion labels<br>

**_Returns_** : <br>
_img_data_ -- array representing images, of shape (Number of images, pixels, pixels)<br>
_labels_ -- list of index values(emotion_code) associated with each image<br>
_img_names_ -- list of names of image files<br>

## `Eyes' Centers :function_1 used in preprocessing`
_Function:_ `eye_centers(landmark)`<br>
__To find the centers of the eyes in the image__<br>

**_Input_** : <br>
_landmark_ --- 68 landmarks detected on the face image using extract_face_landmarks <br>

**_Returns:_** `(point1, point2)`<br>
_point1_ -- center of right eye :mean of 6 landmark points around the right eye: 36-41<br>
_point2_ -- center of left eye :mean of 6 landmark points around the left eye 42-47<br><br>

## `Angle of Rotation :function_2 used in preprocessing`
_Function:_ `find_angle(point1, point2)`<br>
__To find the angle that the line joining these 2 points makes with the horizontal__<br>
Using the arctangent `math.atan()` function in math library, we calculate the angle in radians<br>
Using the degrees conversion function `math.degrees()` we then convert it into degrees<br>

**_Input_** : <br>
_point1_ -- center of right eye<br>
_point2_ -- center of left eye<br>

**_Returns:_** `(angle_d)`<br>
_angle_d_ -- angle in degrees with the horizontal<br>


## `Image Rotation :function_3 used in preprocessing `
_Function_ : `rotate_image(image,angle)`<br>
**Corrects the tilt of a rotated image given the angle(degrees) to be rotated**<br>

**_Input_** : <br>
_image_ -- image to be rotated in (pixels,pixels) shape<br>
_angle_ -- output of _find angle_ function :angle required to align the image horizontally<br>

**_Returns_**:<br>
Perfectly horizontal image<br>

## `Preprocessing` [Reference repository Link](https://github.com/anas-899/facial-expression-recognition-Jaffe)
_Function_ : `preprocessing(input_images)`<br>
**The preprocessing involves rotation of the image and image cropping.**<br>

**Rotation**
The rotation angle for horizontal alignment is calculated on the basis of the eyes centers and then the image is rorated to mke it horizontal 
* _Sequence of the predefined functions used for Rotation:_ eye_centers(landmarks) -->  find_angle(p1, p2) --> rotate_image(img, angle)

**Cropping**
The forehead region was then removed in such a way that
perpendicular distance from the top side of the cropped image
to the horizontal line connecting the eye centres is 0.6 d (d
is the distance between eye centres),The other three sides of the cropped image are defined by the
coordinates of 1st, 9th, and 17th face landmark points.
* _Sequence of steps used for Cropping:_ eye centers for rotated image using eye_centers(extract_face_landmarks(rot_img)) ---> find d, the distance between the two eye centers--> find d_mid, mid point of the 2 eye centers --> crop_img with img(y_start:y_end,x_start:x_end) where y_start: a coordinate 0.6d distance above from the mid point of line joining both eyes, y__end: 9th landmark, x_start: 1st landmark, x_end: 17th landmark

**_Input_** : <br>
input_images-- array of images of shape (Number of images, pixels, pixels)<br>

**_Returns_**:<br>
_preprocessed_faces_ -- array of images after preprocessing (Number of images, pixels, pixels)

## `Normalization` 
_Function:_ `normalization(imagedata, mean, std_dev)`<br>
__To apply Histogram equalization and Z-Square Normalization to the preprocessed images__
We applied the opencv function `cv2.equalizeHist()` on the pre-processed images for histogram equalization<br>
Z-score normalisation is done using the simple formula `[(value-mean)/standard_deviation]`<br>
The normalized images are then resized to `48x48` size using `cv2.resize()`<br>
The resized images are then appended into the _normalised_images_ array using `np.append()` function<br>

**_Input_** : <br>
_imagedata_ -- array of preprocessed images of shape (m,h,w)<br>
_mean_ -- mean of imagedata array<br>
_std_dev_ -- standard deviation of imagedata array<br>

**_Returns:_** `(normalised_images)`<br>
_normalised_images_ -- array of normalised and resized images<br>

## `Model Architecture`
_Function:_ ` recog_model(input_shape)`<br>
__Deep Neural Network architecture (CNN - structure) has been used to train our model__<br>
After the Data Augmentation the images have size of (48,48,1). The model is built using keras.<br>
It consists of 2 convolutional layers and 2 Max pooling layers in order conv1 -> maxpool1 -> conv2 -> maxpool2. Convolutional layers have the activation relu, after maxpool2 it is flattened into a 1600x1 dimensional vector and it’s directly connected to output layer with softmax activation.<br>
The hyperparameters in the model are batch size = 16, learning_rate = 0.001, epochs = 80 to 120.<br>
The model uses momentum optimizer and categorical cross entropy as loss function with accuracy as metrics and it is evaluated using a ten fold cross validation method.<br>

**_Input_** : <br>
_inputshape_ -- array of preprocessed images of shape (h,w,1)<br>

__Returns:__ `(model)`<br>
_model_ -- an instance of the compiled model with the architecture decribed<br>

## 10 fold cross validation `Mean Accuracy and Standard Deviation`<br>
|     |JAFFE 7 classes|JAFFE 6 classes|CK+ 7 classes<br>(327)|CK+ 6 classes<br>(309)|JAFFE & CK+<br>6 classes|
|---|---|---|---|---|---|
|__Mean Accuracy__<br>__(Validation Set)__|93.46%|92.34%|90.54%|92.24%|90.45%|
|__Standard Deviation__|7.05|5.66|5.32|4.12|4.41|

## `Cross Dataset Accuracies`<br>
_40.64%_ when __trained on JAFFE__ and __tested on CK+ (309)__<br>
_30.56%_ when __trained on CK+ (309)__ and __tested on JAFFE__
