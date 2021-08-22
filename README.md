

# Panoptic Segmentation using DETR (DEtection TRansformers) 
## - Assignment 1 - Due 21-Aug

# Solutioning

## Approach

Our Solutioning will be Divided into 2 Phases:
1. To Train DETR to Predict Bounding Boxes
2. ADD Panoptic Segmentation Head on DETR to do Segmentation.

## Phase 1:
We will First train DETR to predict the Bounding Boxes for the Construction Classses (~60) and Stuff. DETR is already trained to Predict Stuff so we will use the Pre-Trained model and then train thte model on our dataset. DETR was trained for 300 EPOCS for COCOA dataset that had 91 classes, our dataset has approx. 60 classes so the training will be done for approx. 200 EPOCS.

### Dataset
We have annotated dataset for construction classes. To have the Ground truth for Phase-1 (BBOxes) we will use a Utility to convert Annotations into BBOx (Bounding Boxes) to train DETR for Phase 1.

### Training 
Classification head of the Pre-trained model will be removed and then it will be trained for ~250 EPOCS while checking for mAP to increase and losses to decrease.
 
## Phase 2:

Once DETR is trained on our Custom classes we will ADD the Panoptic Head over the classification head of the trained DETR and then train for Panoptic Segmentation for approx ~25 EPOCS.

### Dataset
We have annotated Dataset, we will pass the custom dataset images to the pre-trained panoptic model to predict the things and stuff. Since the model is NOT trained on our classes it will NOT segment the custom classes.

We take these output images and ADD segmentation based on the annotations we have done for the images.
We will remove the segmentation done by the pre-trained model for the Annotated area and ADD our Segmentation mask.

This will be the Ground Truth for training our Panoptic Head.

### Training
We freeze the DETR weights and train the Segmentation head for 25 EPOCS.



### 1. We take the encoded image (dxH/32xW/32) and send it to Multi-Head Attention (FROM WHERE DO WE TAKE THIS ENCODED IMAGE?)
The Encoded Image is taken from the DETR Encoder Output, by default d=256. 

CNN Backbone of the DETR outputs a Low Resolution Feature Map of h/32 and w/32. This output is matched with the Hidden Dimensions of DETR (256 default) and then sent to Encoder. The Output of the Encoder is Image Features of the same Dimension. This Output is used as Encoded Image (one of the Inputs) to the Multi Head Attention for Panoptic Segmentation.



### 2. We do something here to generate NxMxH/32xW/32 maps. (WHAT DO WE DO HERE?)

We use Multi-head attention layer to generate Attention Scores over the Encoded Image (dxH/32xW/32) for each Object Embedding (N). This results in NxMxH/32xW/32 attention maps.


### 3. Then we concatenate these maps with Res5 Block (WHERE IS THIS COMING FROM?)

This comes from the CNN Backbone of DETR, when the Image is passed by CNN Backbone, Activations from the Intermediate layers (Res 5, Res4, Res3 ad Res 2) are set aside so that it can be used while doing Panoptic Segmentation.

### 4. Then we perform the above steps (EXPLAIN THESE STEPS)

1. DETR Predicts the BBox for thing (your class) and Stuffs in the image.
2. These BBox along with Encoded Image (from the Encoder Output) is sent to the Multi Head Attention Layer.
3. This Layer generates the Attention Maps for the passed Bboxes using Encoded Image.
4. These Attention Maps are concatenated with Res 5 (output of CNN Backbone Intermediate Layer) and various operations are performed to Upsample and Clean the Masks using intermediate layers output of the CNN backbone.
5. Finally we merge the masks by classigying each pixel to the mask with highest Probability.
6. Resultant image is the Panoptic Segmentation of the Original Image with dimension 1/4 of the Original Image.
