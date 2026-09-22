# MRI-Lesion-Segmentation

# Results
This project utilized MONAI's Brain Tumor Image Segmentation (BraTS) model which was pretrained using multimodal MRI images from the BraTS 2018 dataset for volumetric segmentation of brain tumor subregions [1]. The BraTs model was was fine-tuned to detect multiple scelerosis (MS) lesions utilizing the 2015 International Symposium on Biomedical Imaging (ISBI) Dataset for 3 main reasons: (i) This model expected 4 modalities (T1, T1c, T2, FlAIR) which aligned well with the images provided by the ISBI dataset (T1, pd, T2, FLAIR) -- T1c was substituted with PD images; (ii) The BrATS's whole tumour channel output was a suitable structural proxy for MS lesions; and (iii) Given the assignment's time constraint, this was most reproducible and well-documented publicly accessible model.

Dice scores are listed as the primary metric, per the assignment's core requirement; IoU, sensitivity, and specificity were not computed due to time constraints and limited GPU compute availability. Results demonstrate that the pretrained BraTS model, despite being trained on a different pathology (adult glioma) and lacking any exposure to MS lesions, shows meaningful potential for adaptation to this task once fine-tuned. 

**Tables**

Table 1. Comparison of mean baseline vs fine-tuned dice scores:
|  Baseline  | Fine-tuned | 
| ---------- | ---------- | 
|   0.0907   |   0.3890   | 

Table 2. Loss and Dice scores of Fine-tuned Model: All layers frozen except conv_final
| Epoch | Loss   | Dice Score |
|-------|--------|------------|
| 1     | 0.9429 | 0.0011     |
| 2     | 0.9364 | 0.0011     |
| 3     | 0.9370 | 0.0015     |
| 4     | 0.9366 | 0.0015     |
| 5     | 0.9489 | 0.0015     |
| 6     | 0.9306 | 0.0015     |
| 7     | 0.9433 | 0.0017     |
| 8     | 0.9446 | 0.0020     |
| 9     | 0.9354 | 0.0020     |
| 10    | 0.9312 | 0.0022     |

Table 3. Loss and Dice scores of Fine-tuned Model: Encoder Frozen and Decoder Unfrozen
| Epoch | Loss   | Dice Score |
|-------|--------|------------|
| 1     | 0.9026 | 0.4620     |
| 2     | 0.8449 | 0.5258     |
| 3     | 0.8143 | 0.2215     |
| 4     | 0.7385 | 0.2808     |
| 5     | 0.7924 | 0.0526     |
| 6     | 0.7807 | 0.3920     |
| 7     | 0.7288 | 0.1393     |
| 8     | 0.6699 | 0.4342     |
| 9     | 0.6458 | 0.3853     |
| 10    | 0.6240 | 0.5239     |

**Figures**

<img width="420" height="424" alt="image" src="https://github.com/user-attachments/assets/d1242be0-056c-4633-9143-c4dfae7dd925" />

Figure 1. Image of Normalized FLAIR MRI, Ground Truth Mask, and Predicted Lesions using Baseline BraTS Model

<img width="420" height="424" alt="image" src="https://github.com/user-attachments/assets/91e5a2f3-6530-4acb-a183-15cc0b721719" />

Figure 2. Image of Normalized FLAIR MRI, Ground Truth Mask, and Predicted Lesions using Fine-tuned BraTS Model



# Discussion
### What worked
Partial fine-tuning completed by freezing the encoder and unfreezing the decoder demonstrated improvement compared to the baseline BraTS model and attempt 1 (freezing all layers except final conv layers). Dice validation rose from 0.091 (BrATS baseline) compared to a peak of 0.526 at epoch 2. This suggests potential for the BraTs model in MS segmentation where low- to mid- level features may be transferable to lesion identification. 

### What didn't work
A custom preprocessing pipeline was run as an exploratory exercise: skull stripping via HD-BET and registration of FLAIR/T2/PD to native T1 space via ANTs. However, this dataset was abandoned as they did not align with ground-truth lesion masks which was assumed to be provided in MNI152 space (181, 217, 181).  Re-registering outputs to match the masks' space was out of scope given time constraints. We therefore used the dataset's officially released preprocessed volumes (already skull-stripped and registered to MNI152 space, matching the mask space) for subsequent normalization, training, and evaluation.

The first attempt was a more conservative response at fine-tuning the model, freezing only the final convolution layers based on the assumption that the BraTS model had stronger transferability. This was done initially to prevent overfitting/unstable training due to the small training size. However, the minute improvements to loss and dice scores indicated that adapting only the final layer lacks sufficient capacity to translate the signal into accurate lesion boundaries resulting in attempt 2 where the decoder was retrained as well.

Dice Validation remained highly volatile which may simply be attributed to the small training (3 subjects) and validation set (1 subject). Substituting PD for BraT's T1c images is another factor that may have introduced input distribution shifts beyond structural differences between tumour regions and MS lesions.

### Future Work
With more time/data, this model can be retrained and evaluated to provide more stable estimates of test performance as opposed to relying on a single test/validation subject. More time can also be spent to investigate whether more/less layers should be frozen/unfrozen to enhance dice metrics. Perhaps other models such as those trained specifically to MS lesions, one that utilized the same 4 channels provided by the ISBI dataset (T1, T2, PD, FLAIR), or one with 3 channel inputs could also be utilized to avoid the substitution of PD for T1c. Data augmentations would help prevent overfitting and create a more robust model as well. However, this was not implemented due to time constraints. 

### Limitations & Assumptions
This model utilized an extremely small dataset (5 subjects) resulting in a 3/1/1 split between training/validation/testing groups where we assumed equal lesion burden/stratification across subjects. However, a subject with unusually high/low lesion volume could dominate any one split. Hence, validation and testing scores are not reliable or robust measures of performance. A domain gap was also apparent where the BraTS model was trained to identify glioma tumours which are different in size, shape, and composition than MS lesions.

Several limitations are also be attributed to lack of time: 
- Inter-rater disagreement was never examined as this model only used mask 1 as it's ground truth
- Only 2 fine-coding configurations were examined (final-layer-only vs. decoder-unfrozen), no systematic hyperparameters such as number of epochs, learning weight or loss weighting was examined
- IoU/sensitivity/specificity and data augmentations were skipped due to time/compute constraints meaning results can't fully characterize false positive/negative tradeoffs and may have contributed to the noisy epoch-to-epoch validation respectively



[1] https://catalog.ngc.nvidia.com/orgs/nvidia/monaitoolkit/models/monai_brats_mri_segmentation/-?_lr=1

[2] https://iacl.ece.jhu.edu/index.php?title=MSChallenge

