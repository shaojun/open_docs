#### Rename `jpeg` to `jpg` files  **in place** :


```
cvdata_convert --images_dir image_2/ --in_format jpeg --out_format jpg
```

#### Rename `PNG` to `png` files  **in place** :


```
cvdata_convert --images_dir image_2/ --in_format PNG --out_format png
```

#### Convert `png` to `jpg` (vice versa) files  **in place** :


```
cvdata_convert --images_dir image_2/ --in_format png --out_format jpg
```


#### Resize `kitti` images and label files to a  **new folder** :


```
cvdata_resize --input_images image_2/ --output_images resized_image_2 --input_annotations label_2/ --output_annotations resized_label_2 --width 1280 --height 720 --format kitti
```


#### Rename image and label files name for a `kitti` dataset  **in place** :


```
cvdata_rename --images_dir image_2 --annotations_dir label_2 --format kitti --prefix guidOrOtherStr
```
#### Crop objects from kitti dataset, and group them into folders with class name and generate new image files under each folders.

```
cvdata_crop_objects_to_files --images_dir /home/shao/Downloads/project_labeling_door_closedsign_proj-2022_09_20_03_47_17-kitti\ 1.0/default/image_2 --annotations_dir /home/shao/Downloads/project_labeling_door_closedsign_proj-2022_09_20_03_47_17-kitti\ 1.0/default/label_2 --image_ext jpg --output_dir /home/shao/Downloads/project_labeling_door_closedsign_proj-2022_09_20_03_47_17-kitti\ 1.0/default/classification
```

#### Show statistic info for kitti dataset
ONLY for **.JPG** images, and it's a file statistics, that means multiple lines of same class in single label file will be only count as 1.
```
cvdata_analyze --images resized_image_2/ --annotations resized_label_2/ --format kitti
```
