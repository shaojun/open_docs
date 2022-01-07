- Install Datumaro package:

`pip install datumaro[default]`


- Create a datum project with name `datum_test_project` and `cd` into it:


```
datum create -o datum_test_project
cd datum_test_project
```


- Prepare the dataset which export from `CVAT`

 Download it, unzip, for example here so put it into path `Download/project_labeling_door_closedsign_proj-2022_01_05_03_13_01-datumaro 1.0`


- import the `CVAT` dataset into datum project:

    - for import CVAT dataset with  **datumaro 1.0**  format, and naming the dataset with `myDatumDataset` as its source name:

        `datum import -f datumaro -n myDatumDataset ../project_labeling_door_closedsign_proj-2022_01_05_03_13_01-datumaro\ 1.0`

        should see like：

            Source 'myDatumDataset' with format 'datumaro' has been added to the project

    - for import CVAT dataset with  **kitti 1.0**  format, and naming the dataset with `myKittiDataset` as its source name:
    
        `datum import -f kitti -n myKittiDataset ../project_labeling_door_closedsign_proj-2022_01_05_03_13_01-kitti\ 1.0`

    you can remove the source from project with `source name`:

        datum remove myKittiDataset



- Reorgnize data source

the sources may contains different sizes of images, for align their size, need do a `transform with resize`:
 
`datum transform -t resize -- -dw 800 -dh 600`

you may not want to export all classess of objects from sources:

`datum filter -e '/item/annotation[label="cat" or label="dog"]' -m i+a`

- Export dataset

export the sources into a subfolder: `myFinalExportFolder`, and rename all the images with extension name: `.jpg`:

`datum export -o myFinalExportFolder -f kitti -- --save-images --image-ext='.jpg'`



