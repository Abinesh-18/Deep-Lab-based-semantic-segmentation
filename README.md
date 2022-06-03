# Deep-Lab-based-semantic-segmentation
The primary goal is to use any Deep Lab-based semantic segmentation models to perform the segmentation of facial attributes. 

Celeb dataset https://github.com/switchablenorms/CelebAMask-HQ

# Data Generator
The dataset contains annotation masks for 18 different facial attributes (eyes, ears, mouth, etc). Therefore, this is a 19 task segmentation problem (18 plus background). Each image varies in the amount of attributes it has however, such as some face images wearing glasses and others not. Because of this, the dataset is structured where for each image, there are up to 18 annotation mask images included for labelling. For a given training image, the data generator makes a single ground-truth mask based on the collection of available annotation masks. It was tried making this ground-truth mask both integer encoded by class (so BxHxWx1 dimensional where each pixel in the image takes on the values [0,18]), and one-hot encoded (BxHxWx19 dimensional, one-hot encoding per class per channel). Switching between the representation used based on our exploration of loss functions and class imbalance, discussed in the next section. Our model output is fixed to the one-hot encoding scheme (so HxWx19 dimensional where each channel represents a one-hot encoding mask per class).

#  Loss Functions & Class Imbalance Problem
In the first attempt the model is trained using integer-encoded ground-truth with sparse categorical cross-entropy loss. With the output layer in the model being a convolutional layer, prediction values are logits. In Keras, sparse categorical cross-entropy has a from_logits parameter that takes care of argmaxing/framing the predictions to compute the loss.
To combat this class imbalance sample weights were added to the data generator. Per sample, each pixel was weighted based on the total amount of that class’s pixels that appeared in the image. Using cross-entropy (either sparse with integer encoded truth or regular with one-hot), this produced similar accuracy/mean IoU as the unweighted variant and showed no noticeable difference in the visualization. In theory this should’ve pushed the predictions towards a more uniform distribution. Since it was somewhat out of scope for this project, debugging the weights and other formulas of class weighting were not explored.

The second attempt was to drop in a custom loss function that was more tailored towards multiclass segmentation with an imbalance of pixel classes. Two prominent loss functions were studied, Jaccard Distance and Dice Loss. Instead of looking at the distribution of pixels and their class predictions throughout the image like in cross-entropy, both Jaccard Distance and Dice Loss are formulated more as “IoU losses.
