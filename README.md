# Human Activity Recognition on PAMAP2  
## Zero-Shot, Few-Shot, and Transfer Learning Approaches

This repository investigates **Human Activity Recognition (HAR)** on the **PAMAP2 wearable sensor dataset** under **data-scarce learning scenarios**. We systematically compare **Zero-Shot Learning (ZSL)**, **Few-Shot Learning (FSL)**, and **Transfer Learning (TL)** using a consistent experimental setup.

The objective is to understand how different learning paradigms perform when labeled data for certain human activities is unavailable or extremely limited—an important challenge in real-world wearable and sensor-based systems.

---

## Repository Structure

```
pamap2_zsl.ipynb
pamap2_fsl.ipynb
pamap2_transfer_learning.ipynb
```

Each notebook is self-contained and focuses on a single learning paradigm while sharing:
- the same PAMAP2 dataset
- the same seen / unseen activity split
- subject-disjoint train, validation, and test splits
- comparable evaluation metrics and visualisations

---

## 1. pamap2_zsl.ipynb — Zero-Shot Learning (ZSL)

**Goal**  
Evaluate whether unseen human activities can be recognized without using any labeled examples of those activities during training.

**Summary**
- Trained only on seen activities  
- No labeled unseen data at inference time  
- Uses a semantic embedding space  
- Reports seen accuracy, unseen accuracy, harmonic mean (H), and bias diagnostics  

---

## 2. pamap2_fsl.ipynb — Few-Shot Learning (FSL)

**Goal**  
Recognize unseen activities using a small number of labeled examples per class (k-shot).

**Summary**
- Encoder trained on seen activities  
- Encoder frozen during evaluation  
- Prototype-based classification using support/query sets  
- No parameter updates on unseen data  

---

## 3. pamap2_transfer_learning.ipynb — Transfer Learning (TL)

**Goal**  
Adapt a pretrained HAR model to unseen activities by fine-tuning with limited labeled data.

**Summary**
- Encoder initialized from seen-trained model  
- New classifier head for unseen classes  
- Fine-tuning using gradient descent  
- Same evaluation metrics as few-shot learning  

---

## Comparison Summary

| Aspect | Zero-Shot | Few-Shot | Transfer Learning |
|------|----------|----------|------------------|
| Labeled unseen data | None | k samples | k samples |
| Parameter updates | No | No | Yes |
| Overfitting risk | Low | Low | High |
| Performance ceiling | Low | Medium | High |

---

## Dataset
- PAMAP2 Physical Activity Monitoring Dataset  
- Wearable IMU sensor time-series data  
- Subject-disjoint splits to avoid leakage  

---

## Key Takeaways
- ZSL is challenging for sensor-based HAR  
- Few-shot learning offers strong stability with minimal supervision  
- Transfer learning provides the highest performance when data is available  

---

## Intended Use
- Academic research and MSc projects  
- Wearable sensing and HAR experimentation  
- Demonstrating applied ML expertise in time-series modeling
