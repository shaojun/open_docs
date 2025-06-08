# 24.04.13 - train 2 classes classification of eb and non_eb
dataset size:
* eb 2485    
  most copied from 4 classes classification eb class.
* non_eb 2470    
  recently manual picked by qian from latest production collection.
  
![image](https://github.com/shaojun/open_docs/assets/3241829/8b2925ec-432a-4b29-b0e9-177d6ab62533)

> use the default train config of input_image_size: "3,224,224"
> no prune, no retrain

train result:
```
Found 496 images belonging to 2 classes.
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
```

after deployed to triton server, the custom tool test result:
```
Statistics for model: elenet_two_classes_240413_tao, dataset: /home/shao/Downloads/test
     with confid_watch_point: 0
         Class: eb               -> acc: 0.887(221/249), recall: 0.944(221/221+13), avg infer(by_ms): 148.594, avg confid: 0.911, wrong infer: {non_eb->28}  

     with confid_watch_point: 0.5
         Class: eb               -> acc: 0.887(221/249), recall: 0.944(221/221+13), avg infer(by_ms): 148.594, avg confid: 0.911, wrong infer: {non_eb->28}  

     with confid_watch_point: 0.7
         Class: eb               -> acc: 0.795(198/249), recall: 0.965(198/198+7), avg infer(by_ms): 148.594, avg confid: 1.017, wrong infer: {non_eb->28}  

     with confid_watch_point: 0.9
         Class: eb               -> acc: 0.658(164/249), recall: 0.982(164/164+3), avg infer(by_ms): 148.594, avg confid: 1.228, wrong infer: {non_eb->28}  

     with confid_watch_point: 0
         Class: non_eb           -> acc: 0.947(234/247), recall: 0.893(234/234+28), avg infer(by_ms): 149.797, avg confid: 0.918, wrong infer: {eb->13}  

     with confid_watch_point: 0.5
         Class: non_eb           -> acc: 0.947(234/247), recall: 0.893(234/234+28), avg infer(by_ms): 149.797, avg confid: 0.918, wrong infer: {eb->13}  

     with confid_watch_point: 0.7
         Class: non_eb           -> acc: 0.870(215/247), recall: 0.942(215/215+13), avg infer(by_ms): 149.797, avg confid: 1.000, wrong infer: {eb->13}  

     with confid_watch_point: 0.9
         Class: non_eb           -> acc: 0.716(177/247), recall: 0.972(177/177+5), avg infer(by_ms): 149.797, avg confid: 1.214, wrong infer: {eb->13}  
```