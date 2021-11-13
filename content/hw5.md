title: Homework 5
date: 11/14/2021
author: Xige Huang

# Dive into Deep Learning

Using <https://github.com/cliburn/bios-823-2021/blob/main/homework/assignments.md>

The goal of this homework is to train a neural network to classify beetles, cockroaches and dragonflies, using images from <https://www.insectimages.org/index.cfm>.

First we need to load the packages we need, `tensorflow` and `tensorflow_keras`.
```python
import tensorflow as tf
import tensorflow.keras as keras
```
function to read in images,

```python
from tensorflow.keras.preprocessing import image_dataset_from_directory
```

functions to build model,
```python
from keras.models import Sequential
from keras.layers import Dense, Conv2D , MaxPool2D , Flatten , Dropout 
from tensorflow.keras.optimizers import Adam, SGD, RMSprop
from tensorflow.keras import layers
```

and `SHapley Additive exPlanations` to explain the model
```python
import shap
```

and also `numpy` and `pyplot`.


## Read images
The function `image_dataset_from_directory` in keras is especially convinient reading images and labels from the subdirectories.
```python
img_width = 256
img_height = 256
batchSize = 32
```
There is 1019 files and 180 files in the training and testing set, respectively
```python
train_ds = image_dataset_from_directory(
    directory='insects/train/',
    label_mode='categorical',
    batch_size= batchSize,
    image_size=(img_width, img_height))
test_ds = image_dataset_from_directory(
    directory='insects/test/',
    label_mode='categorical',
    batch_size= batchSize,
    image_size=(img_width, img_height))
```

We can see the class names using `.class_names`, which are beetles, dragonflies, and cockroaches.
```python
class_names = train_ds.class_names
print(class_names)
```

## Data augmentation
Then we can define a data augmentation function, which generate random transformations (horizontal flipping or rotations) of the images. The purpose of data augmentation is to artificially introduce sample diversity. Since the training set we have is of fairly large size (1019 files), this step can be skipped.

One thing to note here is that data augmentation is inactive at test time, so the input samples will only be augmented during model fitting, but not during prediction of evaluation.

```python
data_augmentation = keras.Sequential(
    [
        layers.RandomFlip("horizontal"),
        layers.RandomRotation(0.1),
    ]
)
```


## Model spcification (Convolutional Neural Network)

After the `data_augmentation processor`, we include `Rescaling` layer to standardize the data into values of range(0, 1), dividing by 255 because RGB values have range [0, 255].

Here I dropout regularization to prevent overfitting (i.e., to reduce number of filters/layers), where the dropout ignores a random fraction of units in a layer while setting their weights to zero.
I'm not an expert of neural networks, but I've read that the ideal rate for the input and hidden layers is 0.4, and the ideal rate for the output layer is 0.2 so I use those numbers in the model specification.


```python
model = Sequential([
  data_augmentation,
  layers.Rescaling(1./255, input_shape=(img_height, img_width, 3)),
  layers.Conv2D(32, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(64, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Conv2D(128, 3, padding='same', activation='relu'),
  layers.MaxPooling2D(),
  layers.Dropout(0.4),
  layers.Flatten(),
  layers.Dense(256, activation='relu'),
  layers.Dropout(0.2),
  layers.Dense(num_classes, activation='softmax')
])
```


## Train the model

The training data have 3 categories so here I use `categorical crossentropy` as loss function; if there are 2 categories one could use `binary crossentropy` instead, or if 4 one could use `sparse categorical crossentropy`.
              
```python
epochs = 25

model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

history = model.fit(
    train_ds, epochs=epochs, validation_data=test_ds,
)
```

Processing of the training looks something like
![Hi][output]


Where each epoch steps through the entire dataset, and the `32/32` means that the model is stepping through each batch, where we have 32 in total (as we have batchsize to 32; 32*32 = 1024 and we have 1019 images).

## Model evaluation

By running evaluation function,

```python
score = model.evaluate(test_ds, verbose = 0)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```
We have:
Test loss: 0.13097597658634186
Test accuracy: 0.9333333373069763

If we further increase the epoch numbers, the test accuracy may get even smaller.


We can also test the model in classifying random images in the test set.

For example, if we let the model classify a dragonfly image

```python
img = keras.preprocessing.image.load_img(
    "insects/test/dragonflies/5364299.jpg", target_size=(img_width, img_height)
)
img_array = keras.preprocessing.image.img_to_array(img)
img_array = tf.expand_dims(img_array, 0)  # Create batch axis

predictions = model.predict(img_array)
print(
    "This image is %.2f percent bettles, %.2f percent cockroaches, and %.2f percent dragonflies."
    % (predictions[0, 0]*100, predictions[0, 1]*100, predictions[0, 2]*100)
)
```

The output is:
This image is 0.00 percent bettles, 0.00 percent cockroaches, and 100.00 percent dragonflies.


If we let the model classify a beetles image
```python
img = keras.preprocessing.image.load_img(
    "insects/test/beetles/5558585.jpg", target_size=(img_width, img_height)
)
img_array = keras.preprocessing.image.img_to_array(img)
img_array = tf.expand_dims(img_array, 0)  # Create batch axis

predictions = model.predict(img_array)
print(
    "This image is %.2f percent bettles, %.2f percent cockroaches, and %.2f percent dragonflies."
    % (predictions[0, 0]*100, predictions[0, 1]*100, predictions[0, 2]*100)
)
```
yeilds:

This image is 99.48 percent bettles, 0.52 percent cockroaches, and 0.00 percent dragonflies.


We can also plot the accuracy and loss with respect to the number of epochs

![Hi][acc_loss]

and we can see that there is no massive overfitting of the model

## Visualization using `shap`

First convert the training and testing images and labels into arrays.
```python
X_train = np.concatenate([x for x, y in train_ds], axis = 0)
y_train = np.concatenate([y for x, y in train_ds], axis = 0)
X_test = np.concatenate([x for x, y in test_ds], axis = 0)
y_test = np.concatenate([y for x, y in test_ds], axis = 0)
```

Then specify images to explain and plot
```python
explainer = shap.GradientExplainer(model, X_train)
e = explainer.shap_values(X_test[:10])
shap.image_plot([e[i] for i in range(3)], X_test[:3])
```

![Hi][shap]

The plot above shows the explanations for each class on the three predictions of test set. The explanations are ordered for the classes 0-3 going left to right along the rows. Anything colored in red increases the model output (the model is more confident in the classification), while everything colored blue decreases it.



[output]: {static}/images/output.png
[acc_loss]: {static}/images/acc_loss.png
[shap]: {static}/images/shap.png

