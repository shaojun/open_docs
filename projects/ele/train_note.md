# 24.04.13 - train 2 classes classification of eb and non_eb
dataset size:
* eb 2485
  most copied from 4 classes classification eb class.
* non_eb 2470
  recently manual picked by qian from latest production collection.
  
![image](https://github.com/shaojun/open_docs/assets/3241829/8b2925ec-432a-4b29-b0e9-177d6ab62533)

> use the default train config of input_image_size: "3,224,224"
> no prune, no retrain

result:

> Found 496 images belonging to 2 classes.
2024-04-13 07:32:34,895 [INFO] __main__: Calculating per-class P/R and confusion matrix. It may take a while...
Confusion Matrix
[[236  13]
 [ 15 232]]
Classification Report
              precision    recall  f1-score   support

          eb       0.94      0.95      0.94       249
      non_eb       0.95      0.94      0.94       247

    accuracy                           0.94       496
   macro avg       0.94      0.94      0.94       496
weighted avg       0.94      0.94      0.94       496
