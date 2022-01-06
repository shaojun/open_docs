- Install Datumaro package:

`pip install datumaro[default]`


- create a datum project and `cd` into it:


```
datum create -o datum_test_project
cd datum_test_project
```


- prepare the dataset which export from `CVAT`

 download it, unzip and put into `Download/project_labeling_door_closedsign_proj-2022_01_05_03_13_01-datumaro 1.0`


- import the `CVAT` dataset into datum project:

    - for import CVAT dataset with  **datumaro 1.0**  format:

        `datum import -f datumaro ../project_labeling_door_closedsign_proj-2022_01_05_03_13_01-datumaro\ 1.0`

        should see like：

            Source 'source-2' with format 'datumaro' has been added to the project

    - for import CVAT dataset with  **kitti 1.0**  format:
    
        `datum import -f kitti ../project_labeling_door_closedsign_proj-2022_01_05_03_13_01-kitti\ 1.0`